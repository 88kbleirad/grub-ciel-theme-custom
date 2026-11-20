# GRUB Theme — Ciel Mocha

GRUB2 gfxmenu theme cho VM Arch Linux (virt-manager/kvm, UEFI): nền **Ciel** (nhân vật + khung HUD
+ avatar Ciel nướng sẵn ở góc trên phải), bảng màu **Catppuccin Mocha**, font **BigBlueTerm437 Nerd Font**.

## Preview

<img src="assets/grub-ciel-vm-final6.png" alt="GRUB boot screen" width="720">

## Cấu trúc

```
src/catppuccin-mocha-grub-theme/
├── theme.txt                   # cấu hình theme
├── background.png              # 1920x1080, PNG24 8-bit, non-interlaced (nhân vật + khung HUD + avatar)
├── ciel_bg_grub_edited_3.jpg   # bản export gốc từ Figma (1922x1083 — crop +1+1 trước khi dùng)
├── item_c.png                  # thanh option chưa chọn  rgba(30,30,46,0.65)
├── select_c.png                # thanh option đang chọn  rgba(180,190,254,0.30)
├── logo_ciel.png               # 226x226 PNG32, lavender #B4BEFE (không dùng — avatar đã nướng trong nền)
├── bigblue28.pf2               # font BigBlueTerm437 Nerd Font Regular 28
├── bigblue24.pf2               # font BigBlueTerm437 Nerd Font Regular 24 (menu item)
├── font.pf2                    # font cũ của upstream (không dùng)
└── icons/                      # icon hệ điều hành cho menu entry
```

## Cài đặt

1. Copy theme vào `/boot/grub/themes/` (thay `TEN_THEME` bằng tên thư mục bạn muốn):

   ```sh
   sudo cp -r src/catppuccin-mocha-grub-theme /boot/grub/themes/TEN_THEME/
   sudo cp src/catppuccin-mocha-grub-theme/bigblue28.pf2 /boot/grub/fonts/
   sudo cp src/catppuccin-mocha-grub-theme/bigblue24.pf2 /boot/grub/fonts/
   ```

2. Sửa `/etc/default/grub`:

   ```sh
   GRUB_THEME="/boot/grub/themes/TEN_THEME/theme.txt"
   GRUB_FONTS="bigblue28 bigblue24"
   GRUB_GFXMODE=1920x1080x32,auto
   GRUB_TIMEOUT=30
   # KHÔNG đặt GRUB_TERMINAL_OUTPUT — nó vô hiệu hoá khối gfxterm + theme
   ```

3. Cập nhật cấu hình:

   ```sh
   sudo grub-mkconfig -o /boot/grub/grub.cfg
   ```

   Phải thấy `Found theme: ...`; kiểm tra lại:
   `grep -E "gfxmode|set theme|terminal_output gfxterm" /boot/grub/grub.cfg`

## Bố cục (theme.txt)

| Thành phần | Giá trị |
|---|---|
| `desktop-image` | `background.png` — nướng sẵn nhân vật + khung HUD + avatar, `stretch` |
| `boot_menu` | `left 49%`, `top 27%`, `width 31%`, `height 392` |
| item | `item_height 64`, `item_spacing 12`, `item_padding 12`, `icon_width 36`, `icon_height 36`, `item_icon_space 10`, `scrollbar false` |
| số dòng hiện | **5 dòng** (cửa sổ cuộn trên 10 mục); `rows = (H + spacing − 2*padding) / (item_height + spacing)` ⇒ 5 khi `H = 392` (`5*64 + 4*12 + 2*12`). `H = 368` chỉ cho 4 dòng |
| font | `BigBlueTerm437 Nerd Font Regular 24` cho `item_font`/`selected_item_font`; `…28` cho label; `…26` cho `terminal-font` |
| icon | `icons/<class>.png` 36×36 PNG32; GRUB lấy class đầu tiên có file khớp (`arch`, `windows`, `ubuntu`, `fedora`, `debian`, `opensuse`, `void`, `nixos`, `systemrescue`, `gnu-linux`, `unknown`) |
| màu chữ | `#cdd6f4` (Catppuccin Mocha text) |
| nền thanh | `item_pixmap_style = "item_*.png"` / `selected_item_pixmap_style = "select_*.png"` (bắt buộc có `*` — GRUB 2.16 trả lỗi pattern thiếu `*` → box rỗng) |
| `label` | đồng hồ đếm ngược ở góc dưới-trái (`top 88%`, `left 3%`) |

Thanh option trải x 952…1520: viền trái HUD ở 786…825, dải chấm/glyph trang trí 848…912 (icon bắt đầu
952), avatar bắt đầu ≈1532 (cách 12px — không chạm). Menu dọc: **5 thanh** cửa sổ từ y 315 đến 660
(pitch 76, thanh cao 64px). Viền trên HUD kết thúc 178, hoa văn dưới bắt đầu 805 — cả hai đều **0 px đè**.

10 mục cuộn qua cửa sổ cố định: bấm ↓ tới mục cuối, list hiển thị mục 6…10, mục chọn dừng hàng đáy
(y 607…670); bấm ↑ thì thanh chọn chạy lên (y 379…442) trong cửa sổ đứng yên. Icon 36×36 hiển thị cho
5 hàng nhìn thấy (kiểm chứng pixel). Viewport chữ 547px — tên dài nhất "Advanced options for Arch Linux"
(496px @24) + icon 46px vừa khít, không cắt.

## Lưu ý GRUB 2.16 (đã kiểm chứng)

- `grub-mkfont` không có `--info`; tên font trong `PF2NAME` phải khớp chính xác (vd `BigBlueTerm437 Nerd Font Regular 24`).
- Icons: file phải tên đúng class (`icons/arch.png`), PNG32, `icon_width`/`icon_height` khai báo
  trong `boot_menu`. GRUB lấy icon của **class đầu tiên** trong `--class` có file khớp; tồn tại
  `unknown.png` để dự phòng; `submenu` dùng `submenu.png`.
- Thời `bigblue<size>.pf2` vào `GRUB_FONTS` (vd `"bigblue28 bigblue24 bigblue26"`) để `grub-mkconfig`
  tự `loadfont`; nếu không, GRUB rơi về font mặc định và chữ render to hơn font khai báo.
- Pixmap style phải chứa `*` (`item_*.png`); `"item"` không có `*` → `GRUB_ERR_BAD_ARGUMENT`, thanh option không vẽ.
- `height` boot_menu điều khiển số dòng hiển thị: công thức đầy đủ
  `rows = (height + item_spacing − 2*item_padding) / (item_height + item_spacing)`.
- Chữ dài hơn viewport sẽ bị cắt im ở viền phải (không tự wrap). GRUB chừa sẵn 36px cho icon
  (`icon_width + item_icon_space`); đặt `icon_width = 0` + `item_icon_space = 0` để lấy lại 36px đó
  khi theme không vẽ icon.
- `GRUB_TERMINAL_OUTPUT` phải unset để theme render.

## Kiểm chứng

Đã boot thật trên VM (QEMU/libvirt, `-vga std`, 1920x1080) với **10 mục menu** (Arch, Advanced options,
Windows 11, Ubuntu, SystemRescue, Fedora, Debian, openSUSE, Void, NixOS) và **cửa sổ 5 dòng**:
render 70k+ màu, icon 36×36 hiển thị cho cả 5 hàng nhìn thấy, không đè viền trên / dải chấm / logo /
hoa văn dưới (0 px mỗi vùng), tất cả tên vừa (kể cả "Advanced options for Arch Linux"), cuộn đúng
(↓ dần hiện mục 6…10, mục chọn dừng hàng đáy y 607…670; ↑ chạy lên y 379…442 trong cửa sổ đứng yên),
label đếm ngược hiển thị. Ảnh: `assets/grub-ciel-vm-5rows-icons.png` (đầu) và
`assets/grub-ciel-vm-5rows-icons-scrolled.png` (cuộn tới cuối).
