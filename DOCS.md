# NEON SERPENT 3D — Tài liệu game

> Đọc file này là hiểu toàn bộ game + biết chỗ nào để sửa. Toàn bộ game nằm trong **1 file duy nhất**: [`index.html`](index.html).

---

## 1. Tổng quan & Context

- **Thể loại:** Rắn ăn mồi (Snake) 3D, phong cách "luxury jade" — sang trọng, dịu mắt, neon nhẹ.
- **Mục tiêu:** ăn mồi để dài ra + lên điểm + lên cấp; mỗi cấp rắn **lột xác** sang đẳng cấp cao hơn; nhận **xu** dồn vào ví trên hồ sơ. Đụng tường / thân / đá là chết (hiệu ứng vỡ đầu, chảy máu, rã xác).
- **Triết lý thiết kế:** không chói (bloom yếu, ngưỡng cao), màu cao cấp, chữ rõ. Xem [§9 Quy ước thiết kế].

### Tech stack (zero build)
- **Three.js r0.169** nạp qua `<script type="importmap">` từ CDN jsDelivr — **không cần npm install**.
- Addons: `EffectComposer`, `RenderPass`, `UnrealBloomPass`, `OutputPass`, `RoundedBoxGeometry`, `RoomEnvironment` (PMREM cho phản chiếu PBR).
- Vanilla JS thuần trong 1 `<script type="module">`, không framework. UI là HTML/CSS overlay.
- Âm thanh: Web Audio API synth (không file ngoài).
- Lưu trữ: `localStorage`.

### Chạy / Deploy
- **Phải chạy qua HTTP server** (ES module + importmap bị chặn ở `file://`).
  - `cd D:\game-ai` → `python -m http.server 8777` → mở `http://localhost:8777/index.html`.
- Deploy: upload mỗi `index.html` lên static host bất kỳ (Netlify/Vercel/GitHub Pages/Cloudflare Pages).
- Repo private: `github.com/QuangLe1997/neon-serpent-3d` (nhánh `main`).

---

## 2. Cấu trúc file `index.html`

Thứ tự các phần (tìm bằng comment `// ----- TÊN -----`):

| Vùng | Nội dung |
|---|---|
| `<style>` | Toàn bộ CSS: HUD, overlay, intro, menu, wallet, coin, profile, mobile D-pad, game-over, responsive |
| HTML body | `#hud`, `#wallet`, `#popups`, `#vignette`, `#flash`, `#touch` (D-pad), các overlay: `#intro` `#menu` `#pause` `#gameover` `#profile` |
| CONFIG | `DIFF`, `COMBO_*`, `GOLD_LIFETIME`, `CELL`, `COL`, `EVO` |
| STATE | object `S` (toàn bộ trạng thái runtime) + load localStorage |
| THREE SETUP | renderer, scene, fog, PMREM env, camera, composer/bloom, lights |
| ARENA | `buildArena()` — sàn, lưới, tường rào |
| SNAKE MESHES | `segGeo`, `segMaterial()`, `applyEvo()`, `tuneSegMat()`, `colorForIndex()`, `headDeco` (mắt) |
| FOOD/GOLD/ROCK | mesh mồi, mồi vàng, `makeRockGeo()`, shockwave pool, blood splatter |
| PARTICLES | `burst()`, `trailEmit()`, ambient `dust` |
| AUDIO | `audio()`, `note()`, `sfx()` |
| GAME FLOW | `resetGame()`, `startGame()`, `gameOver()`, `levelUp()`, `moltTo()` |
| TICK | `tickStep()` — logic lưới mỗi nhịp |
| `onEat()` | điểm + combo + xu + lên cấp |
| RENDER | `updateVisuals()` — vẽ/animate mỗi frame |
| HUD/OVERLAY | `updateHUD()`, `popCombo()`, `showLevelBanner()`, `floatPop()`, `countUp()` |
| COIN/WALLET | `awardCoins()`, `addToWallet()`, `renderWallet()`, `openProfile()`, `rankFor()` |
| INPUT | bàn phím, swipe, nút, intro dismiss, mobile D-pad |
| MAIN LOOP | `loop()` — fixed-tick + interpolation + camera |

---

## 3. Vòng đời / State machine

`S.mode` ∈ `intro → menu → playing → (paused) → gameover → menu/playing`

- **intro**: màn cinematic mở đầu (logo, vòng tròn lan, "CHẠM ĐỂ VÀO GAME"). Tự chuyển sau 6.5s hoặc khi chạm/phím. → `menu`.
- **menu**: chọn độ khó + nút BẮT ĐẦU + HỒ SƠ & VÍ. Có rắn idle bò nền (`idleDrift()`), camera orbit nhẹ.
- **playing**: game chạy. Loop dùng accumulator fixed-tick.
- **paused**: Space / nút ⏸. Loop dừng tick, vẫn render.
- **gameover**: chuỗi chết → sau 1.4s hiện overlay.

---

## 4. Gameplay & quy tắc

- **Lưới NxN** (kích thước theo độ khó). Ô = `CELL` (1 đơn vị). `worldPos(cell,y)` đổi ô → toạ độ 3D.
- **Di chuyển:** tick-based. Mỗi nhịp `S.tick` ms rắn tiến 1 ô. Giữa các nhịp, mesh **nội suy tuyến tính** (`t = accumulator/tick`) cho mượt.
- **Hướng:** `S.dir` / `S.nextDir`. Không cho quay đầu 180°.
- **Chết khi:** đầu ra ngoài biên / trùng ô chướng ngại (đá) / trùng thân (trừ chóp đuôi khi không dài ra). → `gameOver()`.
- **Ăn:** đầu trùng ô mồi → dài ra (không bỏ đuôi) + `onEat()`.

---

## 5. Cấp độ & Độ khó (`DIFF`)

| Mode | grid | baseTick | minTick | tickStep | obstPerLvl | goldChance | foodPerLvl |
|---|---|---|---|---|---|---|---|
| easy (DỄ)      | 24 | 200 | 115 | 6  | 0 | 0.16 | 5 |
| normal (THƯỜNG)| 20 | 165 | 82  | 8  | 1 | 0.18 | 5 |
| hard (KHÓ)     | 18 | 138 | 62  | 9  | 2 | 0.20 | 4 |
| insane (ĐIÊN)  | 16 | 112 | 48  | 10 | 3 | 0.22 | 4 |

- **Lên cấp:** mỗi khi ăn đủ `foodPerLvl` mồi → `levelUp()`.
- **Tốc độ (tension):** `S.tick = max(minTick, baseTick − (level−1) × tickStep)`. tick nhỏ = nhanh hơn.
- **Lên cấp thì:** tăng tốc + rơi thêm `obstPerLvl` cục đá + **lột xác** (`moltTo`) + banner + flash + âm thanh.

---

## 6. Tính điểm, Combo & Xu

- **Combo:** ăn liên tiếp trong `COMBO_WINDOW` (2600ms) → `S.combo++` (tối đa `COMBO_MAX`=8). Hệ số `mult = 1 + combo×0.15`.
- **Điểm:**
  - Mồi thường: `round(10 × level × mult)`
  - Mồi vàng: `round((50×level + round(remain×50)) × mult)` (`remain` = % thời gian còn lại của mồi vàng)
- **Xu (coins):**
  - Mồi thường: `3 + combo`
  - Mồi vàng: `25 + round(remain×15) + combo×2`
- **Ví:** `awardCoins()` bắn sprite xu (tối đa 12) **bay đường thẳng tuyến tính** vào chip ví (góc trên phải); mỗi xu rơi vào gọi `addToWallet()` → cộng dồn, đếm vọt (count-up trong loop), nảy chip, lưu localStorage.

---

## 7. Tiến hóa / Lột xác (`EVO`) — 7 bậc

Mỗi lần `levelUp` → `moltTo(min(level−1, 6))`. `applyEvo(tier)` đổi màu (head/body/tail) + chất liệu (metalness, roughness, clearcoat, sheen, iridescence, emissive) cho toàn thân.

| Cấp | tier | Tên | Phong cách |
|---|---|---|---|
| 1 | 0 | LỤC BẢO   | ngọc lục bảo (khởi đầu) |
| 2 | 1 | LAM NGỌC  | lam ngọc kim loại |
| 3 | 2 | TỬ TINH   | tím ánh ngũ sắc |
| 4 | 3 | HỒNG NGỌC | hồng ngọc bóng |
| 5 | 4 | HOÀNG KIM | vàng kim loại |
| 6 | 5 | KIM CƯƠNG | trắng pha lê, iridescence cao |
| 7 | 6 | THẦN LONG | vàng-cam rực rỡ, phát sáng mạnh |

- **FX lột xác** (`moltTo`): vảy cũ bắn ra (particle màu cũ) + shockwave + chùm sáng màu mới + sóng sáng quét head→tail (`moltK`/`moltPulse` trong `updateVisuals`) + âm `molt`.
- Bậc cao nhất từng đạt lưu ở `ns_evo`, hiện trong Hồ sơ.

---

## 8. Vật phẩm & Hiệu ứng

- **Mồi thường:** đá quý san hô (Icosahedron physical, clearcoat) + hào quang + point light. Lơ lửng + xoay.
- **Mồi vàng (bonus):** kim cương vàng, sống `GOLD_LIFETIME` (6200ms), có **vòng đếm ngược** + cảnh báo `#goldwarn` khi sắp hết. Xuất hiện theo `goldChance` sau khi ăn mồi thường.
- **Đá chướng ngại (HAZARD):** `makeRockGeo()` — đa diện méo mó, **xám matte không phát sáng** (rõ ràng khác quà). **Rơi từ trời** (`ROCK_DROP=13` → `ROCK_Y=0.46` trong `ROCK_FALL=520`ms), đáp xuống → `onRockLand()`: **rung màn hình** + bụi + tiếng "BÙM".
- **Hiệu ứng chung:** bloom nhẹ (UnrealBloom strength 0.5, threshold 0.82), `burst()` particle, `shockwave()` ring pool, `dust` bụi không gian, vệt sáng đuôi `trailEmit()`, screen shake (`S.shake`), `floatPop()` (+điểm bay lên), `flash()`, `#vignette`.

### Chết (gore) — `gameOver()` + nhánh `dead` trong `updateVisuals`
1. Đầu vỡ: tiếng "rắc", shockwave đỏ, máu phun (`burst` đỏ nhiều đợt), `bloodSplatter()` vũng máu loang trên sàn.
2. Thân rã: mỗi đốt nhận `S.debris[i]` {vel, ang, s} → văng + trọng lực + nảy sàn + xoay + **co nhỏ & mờ dần** (opacity→0).
3. Camera lao tới xác + rung + vignette đỏ.
4. Sau 1.4s → `showGameOver()` (overlay + count-up điểm/xu).

---

## 9. Quy ước thiết kế (giữ khi sửa)

- **Không chói:** giữ bloom strength thấp (~0.5) + threshold cao (~0.82); emissive vật thể thấp (đá KHÔNG emissive). Chỉ điểm sáng mới glow.
- **Bảng màu `COL`:** mint `#54d6b2`, gold `#e7c06a`, rose `#e493ac`, cream `#f0ead8`. Nền `#0a0e14`.
- **Đá phải xấu/xám** để phân biệt với quà phát sáng.
- **Chữ UI:** tương phản cao, ít text-shadow, panel kính tối.
- File phải **tự chạy** (CDN), không thêm bước build.

---

## 10. State `S` (các field chính)

```
mode, diff, cfg, grid
score, level, best, foodEaten, foodThisLevel
tick, accumulator, combo, lastEat
snake[], prevSnake[], dir, nextDir
food, gold, obstacles[]
shake, deathStart, deathPos, debris
evo, evoColors, evoMat, moltUntil
wallet, runCoins, gamesPlayed, lifeFood, bestEvo
```

## 11. localStorage keys
| key | ý nghĩa |
|---|---|
| `neon_serpent_best` | điểm kỷ lục |
| `ns_wallet` | tổng xu trong ví |
| `ns_games`  | số ván đã chơi |
| `ns_food`   | tổng mồi đã ăn |
| `ns_evo`    | bậc tiến hóa cao nhất (index) |

## 12. Điều khiển
- **Desktop:** mũi tên / WASD di chuyển · Space tạm dừng · R chơi lại.
- **Mobile:** D-pad (góc dưới phải, to) · vuốt toàn màn hình · nút ⏸ / ↻ (góc trên trái). Tự nhận cảm ứng → thêm class `body.touch`.

---

## 13. Cách test nhanh (đã dùng)
- Chạy server rồi mở trên Chrome local. Có thể test headless qua **CDP** (Chrome DevTools Protocol):
  - `chrome --headless=new --remote-debugging-port=PORT --use-gl=swiftshader URL`
  - Kết nối WebSocket devtools → `Runtime.evaluate` để bấm nút, `Page.captureScreenshot` để chụp, bắt `Runtime.exceptionThrown` để soi lỗi JS.
- Để test hàm nội bộ (module scope) tạm thêm `window.__t={...}` ở cuối module, test xong **xoá đi**.
- Luôn kiểm tra cú pháp: trích `<script type="module">` ra file `.mjs` rồi `node --check`.

---

## 14. Ý tưởng mở rộng (chưa làm)
- Cửa hàng đổi skin rắn bằng xu · Bảng xếp hạng online · Nhạc nền intro/menu · Slow-motion lúc va chạm · Telegraph bóng đổ trước khi đá rơi · Bậc cao mọc gai/sừng rồng · Vệt đuôi đổi màu theo bậc.
