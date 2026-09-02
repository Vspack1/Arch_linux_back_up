# HyDE Customization — Wisadel

Ghi chú tất cả các thay đổi / config đã tùy chỉnh trên hệ thống HyDE/Hyprland của Wisadel.
Folder này dùng để up lên GitHub làm tài liệu + backup.

---

## Mục lục

1. [Hyprland keybinds & config](#1-hyprland-keybind-config)
2. [Dunst OSD notifications](#2-dunst-osd-notifications)
3. [Waybar — click workspace & battery](#3-waybar--click-workspace--battery)
4. [Discord / Vesktop theme](#4-discord--vesktop-theme)
5. [Backup trước khi update HyDE](#5-backup)

---

## 1. Hyprland keybind & config

**File:** `config/hypr/hyprland.lua`
**Vị trí gốc:** `~/.config/hypr/hyprland.lua`

Lưu ý: file này là **user override** — HyDE KHÔNG ghi đè khi update. Chứa toàn bộ config cá nhân (khác bản mặc định HyDE).

### Các thay đổi chính:

**a) Volume keys dùng `volumecontrol.sh` → có OSD notify**
- Lý do: mặc định chỉ hạ/tăng volume không hiện OSD. Dùng `volumecontrol.sh` của HyDE để hiện thông báo volume khi bấm phím.
- Dòng ~305-312:
  ```lua
  hl.bind("XF86AudioRaiseVolume", hl.dsp.exec_cmd(hyde.sh.volumecontrol("-o", "i")), { locked = true, repeating = true })
  hl.bind("XF86AudioLowerVolume", hl.dsp.exec_cmd(hyde.sh.volumecontrol("-o", "d")), { locked = true, repeating = true })
  hl.bind("XF86AudioMute",        hl.dsp.exec_cmd(hyde.sh.volumecontrol("-o", "m")), { locked = true, repeating = true })
  hl.bind("XF86AudioMicMute",     hl.dsp.exec_cmd(hyde.sh.volumecontrol("-i", "m")), { locked = true, repeating = true })
  -- Laptop này dùng F2/F3 để tăng/giảm volume (không cần FN)
  hl.bind("F2", hl.dsp.exec_cmd(hyde.sh.volumecontrol("-o", "d")), { locked = true, repeating = true })
  hl.bind("F3", hl.dsp.exec_cmd(hyde.sh.volumecontrol("-o", "i")), { locked = true, repeating = true })
  ```
  - `F2` = giảm, `F3` = tăng (laptop dùng phím F mà không cần giữ Fn)

**b) Brightness keys**
```lua
hl.bind("XF86MonBrightnessUp",   hl.dsp.exec_cmd("brightnessctl -e4 -n2 set 5%+"), { locked = true, repeating = true })
hl.bind("XF86MonBrightnessDown", hl.dsp.exec_cmd("brightnessctl -e4 -n2 set 5%-"), { locked = true, repeating = true })
```

**c) Monitor config** — laptop, màn eDP-1, scale 1.25:
```lua
hl.monitor({
    output   = "eDP-1",
    mode     = "1920x1080@60",
    position = "0x0",
    scale    = "1.25",
})
```

**d) Keybinds thêm cho app riêng (đang comment ở cuối file):**
```lua
hl.bind("SUPER + D",         hl.dsp.exec_cmd("vesktop"),                     ...)  -- Discord
hl.bind("SUPER + SHIFT + O", hl.dsp.exec_cmd("/home/wisadel/Applications/osu.AppImage"), ...) -- osu!lazer
hl.bind("SUPER + ALT + O",   hl.dsp.exec_cmd("osu-wine"),                    ...)  -- osu!stable
hl.dsp.exec_cmd("fcitx5 -d")  -- khởi động fcitx5 (input method)
```

**e) Touchpad** — natural scroll bật.

> Toàn bộ nội dung chi tiết trong `config/hypr/hyprland.lua`.

---

## 2. Dunst OSD notifications

**File:** `config/dunst/dunst.conf`
**Vị trí gốc:** `~/.config/dunst/dunst.conf`

Với sed đã cấu hình OSD volume hiện khi fullscreen.

### Nội dung đã tùy chỉnh:

```ini
[urgency_low]
    background = "#A3656D80"
    foreground = "#FFCCD2E6"
    frame_color = "#A3656D03"
    icon = "${iconsDir}/Wallbash-Icon/hyprdots.svg"
    timeout = 5

[urgency_normal]
    background = "#6B3A5F80"
    foreground = "#FFCCF2E6"
    frame_color = "#6B3A5F03"
    icon = "${iconsDir}/Wallbash-Icon/hyprdots.svg"
    timeout = 5

# Show dunst notifications on top of fullscreen games (e.g. osu!/Wine)
[global]
    layer = overlay

# Always show the volume/brightness OSD (HyDE Notify) even when a
# fullscreen window is open
[fullscreen_osd]
    appname = "HyDE Notify"
    fullscreen = show
```

**Giải thích:**
- `[fullscreen_osd]` + `fullscreen = show`: cho phép OSD volume (appname "HyDE Notify") hiện ngay cả khi đang có cửa sổ fullscreen (chơi game). Mặc định dunst ẩn notification khi fullscreen.
- `layer = overlay`: hiện notification trên lớp overlay để chắc chắn thấy khi fullscreen.

> Lưu ý: `~/.config/dunst/dunstrc` (dunstrc chữ thường ở `~/.config/dunst/`?) KHÔNG cần sửa — file `~/dunstrc` auto-generated bởi `~/.local/share/wallbash/scripts/dunst.sh`. Sửa file `dunst.conf` này và chạy `hyde-shell wallbash dunst` để apply.

---

## 3. Waybar — click workspace & battery

### a) Click workspace (fix mất chức năng click)

**Vấn đề:** bản `waybar` stable 0.15.0 gửi lệnh `dispatch workspace N` cú pháp cũ mà Hyprland Lua từ chối (`')' expected near 'N'`), nên click vào workspace không chuyển được.

**Giải pháp:** cài `waybar-git` (AUR) vì chứa PR waybar #5013 sửa lỗi này.
- Đang chạy: `v0.15.0-1004-g6d60c8e0`
- Click workspace hoạt động bình thường.

### b) Battery indicator

HyDE cấu hình sẵn module `battery` trong `~/.config/waybar/config.jsonc` (group `pill#right2`: `privacy`, `tray`, `battery`). Pin laptop (BAT0) — **đã chai**, sức khỏe ~3.88% (energy-full 1.63Wh / design 41.99Wh).

**File:** `config/waybar/config.jsonc` — hiện chứa battery trong group `pill#right2`.

> Ghi chú: các module waybar nằm ở `~/.local/share/waybar/modules/` (không phải `~/.config/waybar/modules/`). File `battery.jsonc` ở đó.

---

## 4. Discord / Vesktop theme

Vấn đề: mỗi khi `theme.switch`, **wallbash** sinh `discord.css` theo màu wallpaper rồi ghi đè vào `quickCss.css` của vesktop → màu Discord đổi theo hình nền (khi wallpaper xấu thì Discord xấu theo).

### Các file liên quan:

- **`config/hyde-wallbash/discord.sh`** — script wallbash. Mỗi lần chạy, nó `cp` `~/.cache/hyde/wallbash/discord.css` vào `~/.config/vesktop/settings/quickCss.css` (và các client khác). Đây là nguồn gốc việc Discord bị đổi màu theo wallpaper.
  - Vị trí gốc: `~/.config/hyde/wallbash/scripts/discord.sh`
- **`config/hyde-wallbash/discord.dcol`** — template màu (accentcolor, background...) mà wallbash dùng để render `discord.css`.
  - Vị trí gốc: `~/.config/hyde/wallbash/always/discord.dcol`
- **`config/vesktop/quickCss.css`** — file CSS active của vesktop (Quick CSS). Hiện đang set **Catppuccin frappe**:
  ```css
  /* frappe */
  @import url("https://catppuccin.github.io/discord/dist/catppuccin-frappe.theme.css");
  ```
  - Vị trí gốc: `~/.config/vesktop/settings/quickCss.css`

### Trạng thái hiện tại:
- Đã tự gõ `@import catppuccin-frappe` vào quickCss.css → Discord màu frappe đẹp.
- **Lưu ý:** nếu chạy `theme.switch`, wallbash sẽ ghi đè lại quickCss.css theo màu wallpaper. Muốn cố định frappe cần sửa `discord.sh` / `discord.dcol` (chưa thực hiện).

---

## 5. Backup

**Folder:** `backup/backup-pregitupdate-20260902/`

Backup các config trước khi update HyDE (tạo 2026-09-02). Dùng để khôi phục nếu update ghi đè config đã sửa.

```
backup-pregitupdate-20260902/
├── dunst/
│   └── dunst.conf          # dunst config (OSD fullscreen) trước update
├── hypr/
│   └── hyprland.lua        # hyprland config trước update
└── waybar/
    └── hyprland-workspaces.jsonc  # waybar workspaces trước update
```

### Kết quả update 2026-09-02:
- `hyprland.lua` — KHÔNG bị ghi đè (giữ nguyên, user override) ✓
- `dunst.conf` — bị ghi đè bởi update → **đã khôi phục** từ backup ✓
- `hyprland-workspaces.jsonc` — giữ nguyên ✓

---

## Cleanup (files thừa chờ xóa sau khi up GitHub)

Chờ bạn xác nhận sau khi up git, các mục này sẽ xóa:
- `~/.config/cfg_backups/` — toàn bộ backup theme cũ của HyDE (nhiều bản 260828, 260902...)
- `~/.config/hyde/backup-pregitupdate-20260902/` — backup đã sao vào Change
