# dt-policeradar

Radar bắn tốc độ và đọc biển số cho xe cảnh sát. Radar đo tốc độ xe **phía trước và phía sau**, đọc biển số, khóa kết quả, lưu nhật ký, cảnh báo biển số truy nã (BOLO) và tự khóa khi có xe vượt giới hạn tốc độ.

Bản gốc: Samuel#0008 ([GitHub](https://github.com/Samuels-Development/dt-policeradar)). Bản này đã được Việt hóa và chỉnh sửa giao diện cho server DTEAM.

---

## Cài đặt

1. Đặt thư mục `dt-policeradar` vào `resources`.
2. Thêm `ensure dt-policeradar` vào `server.cfg`, đặt **sau** `ox_lib`.
3. Chỉnh `config.lua` nếu cần, rồi restart resource.

**Yêu cầu:** `ox_lib`.

Radar chỉ mở được trên **xe khẩn cấp (class 18)**. Đổi trong `config.lua` > `RestrictToVehicleClass`.

### Thông báo

Mặc định radar dùng **thông báo của QBCore** (`NotificationType = "custom"`, gửi qua event `QBCore:Notify`), nên giao diện giống mọi thông báo khác trên server.

| Thông báo | Loại |
|---|---|
| Khóa/mở khóa, mở/đóng nhật ký, danh sách truy nã, bảng phím tắt | `primary` |
| Lưu kết quả đo, đặt giới hạn tốc độ | `success` |
| Tự khóa khi có xe vượt giới hạn tốc độ | `error` |

Muốn dùng thông báo riêng của radar (khung xanh giữa màn hình) thì đổi `NotificationType = "native"`. Muốn dùng hệ thống thông báo khác thì sửa hàm `ShowNotification` trong `config.lua`.

---

## Phím tắt

Mọi phím đều phải **giữ Ctrl** rồi bấm phím. Ví dụ: muốn bật radar thì giữ `Ctrl` và bấm `F6`.

| Phím | Chức năng | Lệnh chat |
|---|---|---|
| `Ctrl + F6` | Bật/tắt radar | `/radar` |
| `Ctrl + F7` | Bật/tắt chuột để bấm vào giao diện radar | `/radarInteract` |
| `Ctrl + F9` | Khóa/mở khóa **tất cả** (tốc độ + biển số) | `/radarLock` |
| `Ctrl + N` | Khóa/mở khóa **tốc độ** | `/radarLockSpeed` |
| `Ctrl + M` | Khóa/mở khóa **biển số** | `/radarLockPlate` |
| `Ctrl + J` | Lưu kết quả đo vào nhật ký | `/radarSave` |
| `Ctrl + F10` | Mở/đóng nhật ký | `/radarToggleLog` |
| `Ctrl + F11` | Mở/đóng danh sách truy nã (BOLO) | `/radarToggleBolo` |
| `Ctrl + F12` | Hiện/ẩn bảng phím tắt | `/radarToggleKeybinds` |
| — | Mở/đóng cài đặt radar | `/radarSettings` |
| — | Di chuyển radar | `/radarMoveRadar` |
| — | Di chuyển nhật ký | `/radarMoveLog` |
| — | Di chuyển danh sách truy nã | `/radarMoveBolo` |

- **Lệnh chat** chạy được mà không cần giữ Ctrl.
- **Phím `Ctrl + F6` và `Ctrl + F7`** chỉ có tác dụng khi ngồi trong xe hợp lệ. Các phím còn lại chỉ có tác dụng khi radar đang bật.
- **Bấm F6, F7, F9, F11, J, M không kèm Ctrl** thì các script khác chạy như bình thường (Trung tâm hỗ trợ, nhiệm vụ, nhóm, thị trường, FPS, quần áo, điện thoại). Radar không phản ứng.
- **`Ctrl + N`:** N cũng là phím nói chuyện mặc định của GTA, nên bấm `Ctrl + N` sẽ mở mic trong lúc giữ phím.

### Người chơi tự đổi phím

Vào *Esc > Settings > Key Bindings > FiveM*, tìm các mục bắt đầu bằng **"Radar:"** (ví dụ "Radar: Bật/tắt radar (giữ Ctrl)"). Chỉ đổi được phím chính; phím Ctrl là cố định.

### Admin đổi phím mặc định

Trong `config.lua`:

```lua
KeyModifier = "CTRL",   -- "CTRL", "SHIFT", "ALT" hoặc false (không cần giữ)

Keybinds = {
    ToggleRadar = "F6",  -- để nil nếu không muốn gán phím
    ...
}
```

Phím mặc định trong config chỉ áp dụng cho người chơi **chưa từng** gán phím đó. Người đã vào server giữ phím họ đang dùng.

Nếu đổi `KeyModifier` sang `SHIFT` hoặc `ALT`, phải sửa luôn số `36` (mã phím Ctrl) trong 7 script đang nhường phím cho radar: `dt-supportcenter`, `dt-quest`, `dt-groups`, `dt-sellitem`, `dt-fpsbooster`, `dt-dress`, `lb-phone`. Mã mới là Shift = `21`, Alt = `19`.

---

## Đọc màn hình radar

```
 CÙNG NGƯỢC PHÁT   KHÓA NHANH  ▲      CÙNG NGƯỢC PHÁT   KHÓA NHANH  ▲
    [ 079 ]          [  97 ]   ▼         [ 000 ]          [     ]   ▼
 └─────────────── TRƯỚC ───────────┘   └──────────────── SAU ────────────┘

  [ biển số ]   [ biển số ]                 [ 045 ]
  └─ TRƯỚC ─┘   └── SAU ──┘             └─ TỐC ĐỘ XE ─┘
```

Mỗi ăng-ten (**TRƯỚC** và **SAU**) có hai ô số:

| Ô | Màu | Ý nghĩa |
|---|---|---|
| Ô trái | Vàng | Tốc độ xe mục tiêu đang đo |
| Ô phải | Đỏ | Tốc độ **đã khóa**. Để trống khi chưa khóa. |

Các đèn chữ phía trên:

| Đèn | Sáng khi |
|---|---|
| **CÙNG** | Xe mục tiêu chạy **cùng chiều** với bạn |
| **NGƯỢC** | Xe mục tiêu chạy **ngược chiều** |
| **PHÁT** | Ăng-ten đang bật (tắt trong Cài đặt) |
| **KHÓA** | Tốc độ đang bị khóa |
| **NHANH** | Đang bật giới hạn tốc độ; radar sẽ tự khóa khi có xe vượt |

Mũi tên bên phải: **▲** là xe đang đi xa, **▼** là xe đang lại gần.

Hàng dưới:
- **Biển số TRƯỚC / SAU:** biển số xe gần nhất phía trước và phía sau. Khi khóa biển số, nhãn hiện chữ **KHÓA** màu đỏ.
- **Biển số truy nã:** nếu biển số nằm trong danh sách truy nã, ô biển số **nháy viền đỏ** và có âm báo.
- **TỐC ĐỘ XE:** tốc độ xe của chính bạn.

Đơn vị (MPH/KMH) chỉnh trong `config.lua` > `SpeedUnit`.

---

## Thanh nút

Bấm `Ctrl + F7` để hiện chuột, rồi rê chuột vào radar. Một thanh nút nhỏ hiện ra bên dưới radar, hoặc bên trên nếu radar đang nằm ở nửa dưới màn hình.

| Nút | Chức năng |
|---|---|
| 🔒 | Khóa/mở khóa tất cả |
| 💾 | Lưu kết quả đo vào nhật ký |
| ▦ | Mở/đóng nhật ký |
| ⚠ | Mở/đóng danh sách truy nã |
| ⚙ | Mở/đóng cài đặt radar |

Rê chuột vào biển số rồi bấm biểu tượng sao chép ở góc để chép biển số.

Bấm `Ctrl + F7` lần nữa để tắt chuột và lái xe tiếp.

---

## Cài đặt radar

Mở bằng nút ⚙ trên thanh nút (hoặc `/radarSettings`). Đóng bằng nút **Thoát** hoặc phím **Esc**.

**Ăng-ten trước / Ăng-ten sau** (chỉnh riêng từng ăng-ten):

| Mục | Tác dụng |
|---|---|
| Phát sóng (XMIT) | Bật/tắt ăng-ten. Tắt thì ăng-ten đó ngừng đo. |
| Quét làn: CÙNG / NGƯỢC | Chọn đo xe chạy cùng chiều, ngược chiều, hoặc cả hai (sáng cả hai nút). Phải bật ít nhất một nút. |

**Khác:**

| Mục | Tác dụng |
|---|---|
| Giới hạn tốc độ | Nhập số rồi bấm Enter. Khi có xe vượt giới hạn, radar **tự khóa** và báo hướng, tốc độ, biển số. Để trống là tắt. |
| Bật/tắt radar | Tắt radar. |
| Đặt lại khóa | Xóa tốc độ đã khóa và bật lại giới hạn tốc độ để bắt xe tiếp theo. |
| Khóa tốc độ | Bật/tắt khóa tốc độ. |
| Khóa biển số | Bật/tắt khóa biển số. |
| Hiện phím tắt | Hiện/ẩn bảng phím tắt. |
| Di chuyển radar | Bật chế độ kéo thả radar. |

Giới hạn tốc độ chỉ tự khóa **một lần**. Sau khi đã tự khóa, bấm **Đặt lại khóa** (hoặc mở khóa bằng `Ctrl + F9` / `Ctrl + N`) để radar canh xe tiếp theo.

Các cài đặt trên **không được lưu** khi relog hoặc restart resource; mọi thứ quay về mặc định (bật hết, không có giới hạn tốc độ).

---

## Nhật ký

- **Lưu:** bấm `Ctrl + J` hoặc nút 💾. Mỗi mục lưu thời gian, tốc độ trước/sau, tốc độ đã khóa và biển số.
- **Xem:** bấm `Ctrl + F10` hoặc nút ▦.
- **Xóa:** bấm dấu ✕ trên từng mục.

Nhật ký chỉ nằm trên máy người chơi và **mất khi relog**.

---

## Danh sách truy nã (BOLO)

- **Mở:** bấm `Ctrl + F11` hoặc nút ⚠.
- **Thêm biển số:** bấm dấu **+** ở đầu bảng, nhập biển số rồi bấm Enter (hoặc nút **Thêm**).
- **Xóa:** bấm dấu ✕ cạnh biển số.

Khi radar đọc trúng biển số trong danh sách, ô biển số nháy đỏ và có âm báo.

Danh sách **chỉ có trên máy của bạn**, không chia sẻ cho cảnh sát khác và **mất khi relog**. Script khác có thể thêm biển số qua export (xem bên dưới).

---

## Di chuyển và đổi kích thước

1. Mở cài đặt > bật **Di chuyển radar** (hoặc dùng `/radarMoveRadar`, `/radarMoveLog`, `/radarMoveBolo`).
2. Kéo bảng đến vị trí mong muốn. Kéo mép trái/phải để đổi kích thước (nhật ký và danh sách truy nã kéo được cả bốn mép).
3. Tắt chế độ di chuyển khi xong.

Nhật ký và danh sách truy nã còn có nút di chuyển riêng ở đầu bảng.

Vị trí và kích thước được **lưu lại** trên máy người chơi, giữ nguyên qua các lần vào game.

---

## Dành cho dev: event và export

### Event `dt-policeradar:onPlateScanned`

Phát ra mỗi khi radar đọc được biển số mới (phía client).

| Trường | Kiểu | Ý nghĩa |
|---|---|---|
| `plate` | string | Biển số |
| `plateIndex` | number | Kiểu biển số (0–5) |
| `direction` | string | `"front"` hoặc `"rear"` |
| `vehicle` | number | Entity của xe |

```lua
AddEventHandler('dt-policeradar:onPlateScanned', function(data)
    print(('Đọc được biển %s (%s)'):format(data.plate, data.direction))
end)
```

### Export phía client

| Export | Trả về | Tác dụng |
|---|---|---|
| `addBoloPlate(plate)` | `boolean` | Thêm biển số vào danh sách truy nã (tự viết hoa). `false` nếu đã có hoặc không hợp lệ. |
| `removeBoloPlate(plate)` | `boolean` | Xóa biển số khỏi danh sách truy nã. `false` nếu không tìm thấy. |
| `getBoloPlates()` | `table` | Danh sách biển số truy nã hiện tại. |
| `isRadarEnabled()` | `boolean` | Radar đang bật hay không. |
| `toggleRadar()` | — | Bật/tắt radar (vẫn kiểm tra loại xe). |

```lua
-- Đẩy danh sách biển số truy nã từ MDT sang radar
RegisterNetEvent('mdt:sendBoloToRadar', function(plates)
    for _, plate in ipairs(plates) do
        exports['dt-policeradar']:addBoloPlate(plate)
    end
end)
```
