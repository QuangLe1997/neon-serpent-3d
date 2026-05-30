# NEON SERPENT 3D — Tài liệu game

> Đọc file này là hiểu toàn bộ game + biết chỗ nào để sửa, **không cần scan lại `index.html`**. Toàn bộ game nằm trong **1 file duy nhất**: [`index.html`](index.html) (~2600 dòng).
>
> Số dòng dưới đây là **gần đúng** (có thể lệch vài dòng sau khi sửa) — luôn `grep` tên hàm/marker `// ----- TÊN -----` để định vị chính xác.

---

## 0. TRẠNG THÁI TÍNH NĂNG (đọc cái này trước)

Tất cả các mục dưới đây **đã implement & đã test** (trừ khi ghi rõ). Cột "Ở đâu" = tên hàm / marker để `grep`.

### Gameplay lõi
| Tính năng | Trạng thái | Ở đâu |
|---|---|---|
| Rắn lưới NxN, tick-based + nội suy mượt | ✅ | `tickStep()`, `updateVisuals()` |
| 4 độ khó (DỄ/THƯỜNG/KHÓ/ĐIÊN) | ✅ | `DIFF` (CONFIG) |
| Điểm + Combo (×1→×8) | ✅ | `onEat()` |
| Lên cấp + tăng tốc | ✅ | `levelUp()` |
| 7 bậc tiến hóa / lột xác | ✅ | `EVO`, `applyEvo()`, `moltTo()` |
| Mồi thường + mồi vàng (đếm ngược) | ✅ | `spawnFood()`, `spawnGold()` |
| Đá chướng ngại rơi từ trời | ✅ | `makeRockGeo()`, `onRockLand()` |
| Xu + ví + hồ sơ | ✅ | `awardCoins()`, `renderWallet()`, `openProfile()` |
| Chết: vỡ đầu + máu + rã xác | ✅ | `gameOver()`, nhánh `dying` trong `updateVisuals` |

### Power-ups & Mạng (mới)
| Tính năng | Trạng thái | Ở đâu |
|---|---|---|
| 5 power-up: Khiên/Chậm/X2/+Mạng/Cưa đá | ✅ | `POW_TYPES`, `spawnPowerup()`, marker `POWER-UPS` (~1556) |
| Hệ mạng (3 mạng, hồi sinh giữ độ dài) | ✅ | `S.lives/maxLives`, `gainLife()`, respawn trong `gameOver()` |
| Khiên chặn 1 cú chết + bất tử ngắn | ✅ | `S.shield`, `S.invincibleUntil`, `breakShield()` |
| Cưa đá: phá đá 15s khi sân chật | ✅ | `S.sawUntil` (spawn khi `obstacles>10`) |

### Vũ trụ / Bối cảnh (mới — chỉ desktop, `COSMOS = !MOBILE`)
| Tính năng | Trạng thái | Ở đâu |
|---|---|---|
| Starfield 3 lớp + dải Ngân Hà + sao tĩnh | ✅ | `buildCosmos()` (~730) |
| **Cosmic flyby**: 1 sự kiện/lúc, drift chậm | ✅ | marker `COSMIC FLYBY EVENTS` (~883) |
| 14 biến thể hành tinh/sao (Sao Thổ vành, sao neon, Hỏa, Thủy…) | ✅ | `FLYBY_BODIES`, `makeCelestialBody()`, `buildPlanetFlyby()` |
| Mặt trời dung nham (gradient mượt, halo nhỏ) | ✅ | `sunTexture()`, `buildSunFlyby()` |
| Thiên thạch bay ngang sân (trục Z) + vỡ rơi đá | ✅ | `buildMeteorFlyby()` |
| Nhịp flyby theo mode (menu nhanh, chơi chậm) | ✅ | `scheduleNextFlyby()`, `pickFlyby()` |
| **Menu cosmos**: 6 hành tinh luôn hiện sau tựa đề | ✅ | `buildMenuCosmos()`, `updateMenuCosmos()` (~1200) |
| Rắn demo bò quanh viền sân ở intro/menu | ✅ | `startDemo()`, `dashStep()` (attract branch trong `loop()`) |

### Âm thanh (mới)
| Tính năng | Trạng thái | Ở đâu |
|---|---|---|
| SFX synth (ăn/vàng/chết/lên cấp…) | ✅ | `sfx()`, `note()` |
| Nhạc nền: 2 track synthwave (130/138 BPM), random/phiên | ✅ | `SONGS`, `SONG`, `startMusic()` (ZzFXM) |
| Nhạc tăng tốc theo cấp | ✅ | `updateMusicRate()` |
| iOS/Android: `actx.resume()` trong gesture | ✅ | `audio()` |

### UI / UX (làm lại)
| Tính năng | Trạng thái | Ở đâu |
|---|---|---|
| HUD chip thống nhất (điểm/ví/nút) | ✅ | CSS `#hud-top`,`#wallet`,`.ctrl-btn`,`.mbtn` |
| Menu gọn: tựa đề 1 dòng + pill độ khó | ✅ | HTML `#menu`, CSS `.diff-pills`,`.pill` |
| Game-over: hero score + 3 stat card | ✅ | HTML `#gameover`, `endSession()` |
| Mobile joystick (kéo-lái), tap-leak fixed | ✅ | marker `MOBILE CONTROLS` (~2418), `armMobileBtn()` |
| viewport-fit=cover + safe-area insets | ✅ | `<meta viewport>`, CSS `env(safe-area-inset-*)` |

### Dev helpers (để lại, vô hại — `window._*`)
- `window._forceFlyby('planet'|'sun'|'meteors')` — ép một sự kiện vũ trụ.
- `window._flybyStatus()` — trạng thái flyby hiện tại + vị trí camera.
- `window._menuCosmosDebug()` — toạ độ màn hình (%) của từng hành tinh menu.

### Chưa làm (ý tưởng mở rộng)
Cửa hàng đổi skin bằng xu · Bảng xếp hạng online · Bậc cao mọc gai/sừng rồng.

---

## 1. Tổng quan & Context

- **Thể loại:** Rắn ăn mồi (Snake) 3D, phong cách "luxury jade" — sang trọng, dịu mắt, neon nhẹ; bối cảnh "lạc trôi giữa thiên hà".
- **Mục tiêu:** ăn mồi để dài ra + lên điểm + lên cấp; mỗi cấp rắn **lột xác** sang đẳng cấp cao hơn; nhặt **power-up**; nhận **xu** dồn vào ví. Đụng tường / thân / đá là chết (vỡ đầu, chảy máu, rã xác) — trừ khi còn mạng/khiên.
- **Triết lý thiết kế:** không chói (bloom yếu, ngưỡng cao), màu cao cấp, chữ rõ. Xem [§9 Quy ước thiết kế].

### Tech stack (zero build)
- **Three.js r0.169** nạp qua `<script type="importmap">` từ CDN jsDelivr — **không cần npm install**.
- Addons: `EffectComposer`, `RenderPass`, `UnrealBloomPass`, `OutputPass`, `RoundedBoxGeometry`, `RoomEnvironment` (PMREM), `Reflector`.
- Vanilla JS thuần trong 1 `<script type="module">`, không framework. UI là HTML/CSS overlay.
- Âm thanh: Web Audio API synth + ZzFXM (không file ngoài).
- Lưu trữ: `localStorage`.

### Chạy / Deploy
- **Phải chạy qua HTTP server** (ES module + importmap bị chặn ở `file://`).
  - `cd <repo>` → `python3 -m http.server 8765` → mở `http://localhost:8765/index.html`.
- Deploy: upload mỗi `index.html` lên static host bất kỳ (Netlify/Vercel/GitHub Pages/Cloudflare Pages).
- Repo: `github.com/QuangLe1997/neon-serpent-3d` — nhánh `main` & `dev` (đồng bộ).

---

## 2. Cấu trúc file `index.html`

Thứ tự các phần (tìm bằng comment `// ----- TÊN -----`):

| Marker / vùng | Nội dung |
|---|---|
| `<style>` | Toàn bộ CSS: HUD chip, overlay, intro, menu pill, wallet, coin, profile, joystick, game-over card, responsive |
| HTML body | `#hud` `#hud-top` `#wallet` `#lives` `#buffs` `#popups` `#vignette` `#flash` `#muteBtn` `#inputShield` `#touch`(joystick); overlay: `#intro` `#menu` `#pause` `#gameover` `#profile` |
| CONFIG | `DIFF`, `COMBO_*`, `GOLD_LIFETIME`, `CELL`, `COL`, `EVO` |
| STATE | object `S` (toàn bộ runtime) + load localStorage |
| THREE SETUP | renderer, scene, fog, PMREM env, camera, composer/bloom, lights |
| NEBULA SKYDOME | `nebula` — vòm sao mờ theo camera |
| COSMOS *(desktop)* | `buildCosmos()` starfield/Ngân Hà/sao tĩnh/tiểu hành tinh/sao chổi; `updateCosmos()` |
| COSMIC FLYBY EVENTS | `FLYBY_BODIES`, `makeCelestialBody()`, `buildPlanetFlyby/SunFlyby/MeteorFlyby()`, `scheduleNextFlyby/pickFlyby/startFlyby/updateFlyby()` |
| MENU COSMOS | `buildMenuCosmos()`, `updateMenuCosmos()` — hành tinh luôn hiện ở intro/menu |
| BIOME | `setBiome()`, `applyBiomeNow()`, `updateBiome()` — đổi tông theo bậc tiến hóa |
| ARENA | `buildArena()`, `computeDims()`, `worldPos()` |
| SNAKE MESHES | `segGeo`, `applyEvo()`, `tuneSegMat()`, `colorForIndex()`, `headDeco` (mắt) |
| FOOD/GOLD/OBSTACLES | mesh mồi, mồi vàng, `makeRockGeo()`, shockwave pool, `bloodSplatter()`, `syncObstacleMeshes()` |
| POWER-UPS | `POW_TYPES`, `spawnPowerup()`, `pickupPower()`, `breakShield()`, `gainLife()` |
| PARTICLES | `burst()`, `trailEmit()`, ambient `dust` |
| AUDIO | `audio()` (+resume), `note()`, `sfx()` |
| BACKGROUND MUSIC | `SONGS` (2 track), `SONG`, `zzfxG/zzfxM`, `startMusic()`, `updateMusicRate()` |
| GAME FLOW | `resetGame()`, `startGame()`, `gameOver()`(+respawn), `endSession()`, `levelUp()`, `moltTo()` |
| TICK | `tickStep()` — logic lưới mỗi nhịp |
| RENDER UPDATE | `updateVisuals()` — vẽ/animate mỗi frame |
| HUD/OVERLAYS | `updateHUD()`, `updateLivesHUD()`, `refreshBuffs()`, `showLevelBanner()`, `floatPop()`, `countUp()` |
| COIN/WALLET | `awardCoins()`, `addToWallet()`, `renderWallet()`, `openProfile()`, `rankFor()` |
| INPUT | bàn phím, `togglePause()`, gắn nút overlay |
| INTRO → MENU | `dismissIntro()` |
| MOBILE CONTROLS | joystick (`joyStart/Move/End`), `armMobileBtn()`, touch-guard, input shield |
| IDLE MENU SCENE | `startDemo()`, `dashStep()`, `demoStep()` |
| MAIN LOOP | `loop()` — fixed-tick + interpolation + camera + attract mode |

---

## 3. Vòng đời / State machine

`S.mode` ∈ `intro → menu → playing → (paused) → dying → gameover → menu/playing`

- **intro**: cinematic mở đầu (logo, vòng tròn lan, "CHẠM ĐỂ VÀO GAME"). Tự chuyển sau 6.5s hoặc khi chạm/phím → `menu`.
- **menu**: chọn độ khó (pill) + BẮT ĐẦU + HỒ SƠ. **Rắn bò quanh viền sân** (`dashStep`) + **menu cosmos** (hành tinh) hiện sau tựa đề + camera orbit nhẹ + flyby nhịp nhanh (~12–24s).
- **playing**: game chạy. Loop dùng accumulator fixed-tick. Menu cosmos ẩn đi; flyby nhịp chậm (50–110s).
- **paused**: Space / nút ⏸. Loop dừng tick, vẫn render.
- **dying**: chuỗi chết (vỡ đầu, rã xác) ~1.4s.
- **gameover**: nếu còn mạng → **respawn** (giữ độ dài, bất tử ngắn); hết mạng → overlay `endSession()`.

---

## 4. Gameplay & quy tắc

- **Lưới NxN** (kích thước theo độ khó; portrait mobile = chữ nhật cao). Ô = `CELL` (1 đơn vị). `worldPos(cell,y)` đổi ô → toạ độ 3D.
- **Di chuyển:** tick-based. Mỗi nhịp `S.tick` ms rắn tiến 1 ô; giữa nhịp mesh **nội suy tuyến tính** (`t = accumulator/tick`).
- **Hướng:** `S.dir` / `S.nextDir`. Không cho quay đầu 180°.
- **Chết khi:** đầu ra biên / trùng ô đá / trùng thân — trừ khi **khiên** chặn (`breakShield`) hoặc đang bất tử. Hết mạng → game over.
- **Ăn:** đầu trùng ô mồi → dài ra + `onEat()`.

---

## 5. Cấp độ & Độ khó (`DIFF`)

| Mode | grid | baseTick | minTick | tickStep | obstPerLvl | goldChance | foodPerLvl |
|---|---|---|---|---|---|---|---|
| easy (DỄ)      | 24 | 200 | 115 | 6  | 0 | 0.16 | 5 |
| normal (THƯỜNG)| 20 | 165 | 82  | 8  | 1 | 0.18 | 5 |
| hard (KHÓ)     | 18 | 138 | 62  | 9  | 2 | 0.20 | 4 |
| insane (ĐIÊN)  | 16 | 112 | 48  | 10 | 3 | 0.22 | 4 |

- **Lên cấp:** ăn đủ `foodPerLvl` mồi → `levelUp()`.
- **Tốc độ:** `S.tick = max(minTick, baseTick − (level−1) × tickStep)`. tick nhỏ = nhanh hơn.
- **Lên cấp:** tăng tốc + rơi thêm `obstPerLvl` đá + **lột xác** + banner + flash + âm + nhạc tăng tốc. Mỗi cấp thứ 3 → **mưa thiên thạch** (đá phụ).

---

## 6. Tính điểm, Combo & Xu

- **Combo:** ăn liên tiếp trong `COMBO_WINDOW` (2600ms) → `S.combo++` (tối đa `COMBO_MAX`=8). `mult = 1 + combo×0.15`.
- **Điểm:** mồi thường `round(10×level×mult)`; mồi vàng `round((50×level + round(remain×50))×mult)`.
- **Xu:** mồi thường `3 + combo`; mồi vàng `25 + round(remain×15) + combo×2`.
- **Ví:** `awardCoins()` bắn sprite xu (≤12) bay tuyến tính vào chip ví; mỗi xu rơi vào → `addToWallet()` (cộng dồn, count-up, nảy chip, lưu localStorage).

---

## 7. Tiến hóa / Lột xác (`EVO`) — 7 bậc

Mỗi `levelUp` → `moltTo(min(level−1, 6))`. `applyEvo(tier)` đổi màu (head/body/tail) + chất liệu + **biome** (nền/fog/sàn/lưới/accent đổi theo bậc — xem `setBiome`).

| Cấp | tier | Tên | Phong cách |
|---|---|---|---|
| 1 | 0 | LỤC BẢO   | ngọc lục bảo (khởi đầu) |
| 2 | 1 | LAM NGỌC  | lam ngọc kim loại |
| 3 | 2 | TỬ TINH   | tím ánh ngũ sắc |
| 4 | 3 | HỒNG NGỌC | hồng ngọc bóng |
| 5 | 4 | HOÀNG KIM | vàng kim loại |
| 6 | 5 | KIM CƯƠNG | trắng pha lê, iridescence cao |
| 7 | 6 | THẦN LONG | vàng-cam rực rỡ, phát sáng mạnh |

- **FX lột xác** (`moltTo`): vảy cũ bắn ra + shockwave + chùm sáng màu mới + sóng sáng quét head→tail + âm `molt`.
- Bậc cao nhất lưu ở `ns_evo`, hiện trong Hồ sơ.

---

## 8. Vật phẩm, Power-up & Hiệu ứng

### Mồi & đá
- **Mồi thường:** đá quý san hô (Icosahedron physical) + hào quang + point light. Lơ lửng + xoay.
- **Mồi vàng:** kim cương vàng, sống `GOLD_LIFETIME` (6200ms), có vòng đếm ngược + `#goldwarn`. Theo `goldChance` sau khi ăn mồi thường.
- **Đá chướng ngại:** `makeRockGeo()` — đa diện méo, xám matte (rõ khác quà). Rơi từ trời (`ROCK_DROP=13` → `ROCK_Y=0.46` trong `ROCK_FALL=520`ms) → `onRockLand()`: rung màn hình + bụi + "BÙM".

### Power-up (`POW_TYPES`, `POW_LIFE=7200`, `POW_CHANCE=0.16`)
| id | emoji | Tác dụng |
|---|---|---|
| `shield` 🛡️ | KHIÊN | chặn 1 cú chết, sau đó bất tử ngắn (`invincibleUntil`) |
| `slow` ⏳ | CHẬM | tick ×1.7 (chậm) trong `slowUntil` |
| `x2` 2️⃣ | X2 ĐIỂM | nhân đôi điểm trong `x2Until` |
| `life` ❤️ | +1 MẠNG | hiếm — `gainLife()` (≤ `maxLives`=10) |
| `saw` 🪚 | CƯA ĐÁ | phá đá 15s; chỉ spawn khi `obstacles>10` (sân chật) |

- Buff đang chạy hiển thị ở `#buffs` (chip màu riêng), cập nhật bởi `refreshBuffs()`.

### Mạng & hồi sinh
- Bắt đầu 3 mạng (`S.lives`). Chết khi còn mạng → respawn **giữ nguyên độ dài** (`S.respawnLen`) + bất tử ngắn. Hết mạng → `endSession()`.

### Chết (gore) — `gameOver()` + nhánh `dying` trong `updateVisuals`
1. Đầu vỡ: tiếng "rắc", shockwave đỏ, máu phun nhiều đợt, `bloodSplatter()` vũng loang.
2. Thân rã: mỗi đốt nhận `S.debris[i]` {vel, ang, s} → văng + trọng lực + nảy + xoay + co nhỏ & mờ dần.
3. Camera lao tới xác + rung + vignette đỏ.

---

## 8b. Vũ trụ / Bối cảnh (desktop)

> `COSMOS = !MOBILE`. Mobile tắt cosmos để giữ FPS.

- **Nền tĩnh** (`buildCosmos`): starfield 3 lớp, dải Ngân Hà, mặt trời + vài hành tinh xa, vành tiểu hành tinh, sao chổi thi thoảng. `updateCosmos()` xoay trời rất chậm.
- **Cosmic flyby** (1 sự kiện/lúc, paced): `updateFlyby()` spawn theo `flybyNextAt`. Tỉ lệ `pickFlyby()`: 55% hành tinh / 25% mặt trời / 20% thiên thạch. Đường đi tính theo **camera-relative** (`_camPath`) để luôn lọt khung.
  - **Hành tinh/sao** (`buildPlanetFlyby` + `FLYBY_BODIES`, 14 biến thể): `star` (tự phát sáng, halo to, pulse) / `gas` (to, có vành: Sao Thổ, Hải Vương, Thiên Vương) / `rocky` (Hỏa, Thủy, Kim, Trái Đất…). `makeCelestialBody(v,r)` dựng sphere + ring + halo (dùng chung với menu cosmos).
  - **Mặt trời** (`buildSunFlyby` + `sunTexture`): 1 sprite gradient dung nham đa-stop (trắng→vàng→cam→đỏ→tro), halo nhỏ cỡ hành tinh, ám vàng-đỏ nhẹ lên biome/fog khi tới gần.
  - **Thiên thạch** (`buildMeteorFlyby`): 1 quả lớn bay ngang **trục Z** trên sân (lõi đá xoay + head + tail + ember), **vỡ tại 2–4 điểm** → spawn đá rơi vào ô bên dưới (qua hệ rock-fall) + flash + "thud".
- **Menu cosmos** (`buildMenuCosmos`/`updateMenuCosmos`): 6 thiên thể bố trí sẵn (Sao Thổ vành, sao neon, Hỏa, Hải Vương, sao lùn xanh, Thủy) **luôn hiện ở intro/menu**, đặt theo camera-basis (parallax nhẹ), trôi/bồng bềnh/pulse; **thu nhỏ ẩn khi vào chơi**. Portrait: co khoảng cách ngang + nâng cao để nằm trên panel.

---

## 9. Quy ước thiết kế (giữ khi sửa)

- **Không chói:** bloom strength thấp (~0.5) + threshold cao (~0.82); emissive thấp (đá KHÔNG emissive). Chỉ điểm sáng mới glow.
- **Bảng màu `COL`:** mint `#54d6b2`, gold `#e7c06a`, rose `#e493ac`, cream `#f0ead8`. Nền `#0a0e14`.
- **HUD chip thống nhất:** mọi phần header (điểm/ví/nút) dùng cùng surface gradient + blur(14px) + viền + vạch sáng đỉnh. Giữ ngôn ngữ này khi thêm chip mới.
- **Overlay ẩn phải tắt click:** `.overlay.hidden, .overlay.hidden *{pointer-events:none}` — **đừng bỏ** (nếu bỏ, nút vô hình lại ăn chạm joystick → bug pause/menu).
- **Đá phải xấu/xám**; **mặt trời/halo nhỏ** cỡ hành tinh, không tràn màn hình.
- File phải **tự chạy** (CDN), không thêm bước build.

---

## 10. State `S` (các field chính)

```
mode, diff, cfg, grid, cols, rows
score, level, best, foodEaten, foodThisLevel
tick, accumulator, combo, lastEat, effTick
snake[], prevSnake[], dir, nextDir
food, gold, obstacles[]
powerup, shield, slowUntil, x2Until, sawUntil      // power-ups
grace, invincibleUntil, lives, maxLives, respawnLen // mạng & hồi sinh
shake, fovKick, deathStart, deathPos, debris, freezeUntil, cdNum
evo, evoColors, evoMat, moltUntil, biome, biomeMix  // tiến hóa + biome
wallet, runCoins, gamesPlayed, bestEvo
demoEaten, _nextEvoAt, camTX, camTZ, distMul        // attract mode + camera
```

## 11. localStorage keys
| key | ý nghĩa |
|---|---|
| `neon_serpent_best` | điểm kỷ lục |
| `ns_wallet` | tổng xu trong ví |
| `ns_games`  | số ván đã chơi |
| `ns_food`   | tổng mồi đã ăn |
| `ns_evo`    | bậc tiến hóa cao nhất (index) |
| `ns_muted`  | tắt tiếng (1/0) |

## 12. Điều khiển
- **Desktop:** mũi tên / WASD di chuyển · Space tạm dừng · R chơi lại · nút loa (góc trên trái) bật/tắt tiếng.
- **Mobile:** **joystick kéo-lái** ở giữa-dưới (`#joypad` → grab & drag, 4 hướng, dùng pointer-capture) · nút ⏸/↻ + loa (góc trên trái). Tự nhận cảm ứng → class `body.touch`.
  - **Tap-leak đã fix:** joystick `setPointerCapture` + `touchstart preventDefault` + nuốt `click` capture-phase; overlay ẩn tắt pointer-events. Swipe trên sân **đã tắt** (chỉ joystick + nút + overlay nhận input).
  - **Audio iOS/Android:** `audio()` gọi `actx.resume()` trong mọi user gesture (nếu không sẽ câm).

---

## 13. Cách test nhanh (đã dùng)
- Chạy `python3 -m http.server 8765` rồi mở Chrome local. Test tự động qua **Playwright MCP** (`browser_navigate`, `browser_evaluate`, `browser_take_screenshot`, `browser_resize` để giả mobile 390×844 / desktop 1280×800).
- Ép sự kiện vũ trụ để chụp: `window._forceFlyby('planet'|'sun'|'meteors')`, soi vị trí `window._flybyStatus()` / `window._menuCosmosDebug()`.
- Để test hàm module-scope: dùng `window._*` helper sẵn có; thêm tạm thì **xoá sau**.
- Kiểm cú pháp: trích `<script type="module">` ra `.mjs` rồi `node --check`.

---

## 14. Lịch sử cập nhật lớn
- **2026-05-31** (commit `fac61a4`): menu cosmos + rắn bò viền; trước đó cùng đợt: cosmic flyby (14 hành tinh/sao), mặt trời dung nham, thiên thạch bay ngang vỡ rơi đá, HUD chip, menu/game-over làm lại, 2 track nhạc synthwave, fix tap-leak + audio iOS, viewport safe-area.
- Trước đó: power-up cưa đá, hệ mạng giữ độ dài khi respawn, cosmos nền (starfield/Ngân Hà/sao/tiểu hành tinh/sao chổi).

> **Last updated:** 2026-05-31 · nhánh `main`/`dev` @ `fac61a4`
