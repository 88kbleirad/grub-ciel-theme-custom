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
| `boot_menu` | `left 49%`, `top 16%`, `width 30%`, `height 640` |
| item | `item_height 58`, `item_spacing 4`, `item_padding 12`, `icon_width 0`, `item_icon_space 0`, `scrollbar false` |
| số dòng hiện | 10 dòng; `rows = (H + spacing − 2*padding) / (item_height + spacing)` ⇒ 10 khi `H = 640` (`10*58 + 9*4 + 2*12`) |
| font | `BigBlueTerm437 Nerd Font Regular 26` cho `item_font`/`selected_item_font` + `terminal-font`; `…28` cho label |
| màu chữ | `#cdd6f4` (Catppuccin Mocha text) |
| nền thanh | `item_pixmap_style = "item_*.png"` / `selected_item_pixmap_style = "select_*.png"` (bắt buộc có `*` — GRUB 2.16 trả lỗi pattern thiếu `*` → box rỗng) |
| `label` | đồng hồ đếm ngược ở góc dưới-trái (`top 88%`, `left 3%`) |

Thanh option trải x 952…1501: viền trái HUD ở 786…825, dải chấm/glyph trang trí 848…912 (chữ bắt đầu
952 nên **không đè** dải), avatar bắt đầu ≈1532 (cách 31px — không chạm).
Menu dọc: 10 thanh từ y 184 đến 799 (pitch 62). Viền trên HUD kết thúc 178, hoa văn dưới bắt đầu 805 —
cả hai đều **0 px đè** (list trải kín vùng trong của khung).

Cuộn dùng `make_selected_item_visible` của GRUB: khối menu **không di chuyển**, chỉ `first_shown_index`
đổi. Với 10 option, bấm ↓ tới option cuối thì danh sách vẫn dừng ở hàng đáy (y 742…799, cách hoa văn 6px —
không trượt, không dư khoảng trống); bấm ↑ thì chỉ thanh chọn chạy lên trong khối đứng yên.
Viewport chữ 525px — tên dài nhất "Advanced options for Arch Linux" (537px @26, ink 525px) vừa khít, không cắt.

## Lưu ý GRUB 2.16 (đã kiểm chứng)

- `grub-mkfont` không có `--info`; tên font trong `PF2NAME` phải khớp chính xác (vd `BigBlueTerm437 Nerd Font Regular 26`).
- Thời `bigblue26.pf2` vào `GRUB_FONTS` (vd `"bigblue28 bigblue24 bigblue26"`) để `grub-mkconfig`
  tự `loadfont` nó; nếu không, GRUB rơi về font mặc định và chữ render to hơn font khai báo.
- Pixmap style phải chứa `*` (`item_*.png`); `"item"` không có `*` → `GRUB_ERR_BAD_ARGUMENT`, thanh option không vẽ.
- `height` boot_menu điều khiển số dòng hiển thị: công thức đầy đủ
  `rows = (height + item_spacing − 2*item_padding) / (item_height + item_spacing)`.
- Chữ dài hơn viewport sẽ bị cắt im ở viền phải (không tự wrap). GRUB chừa sẵn 36px cho icon
  (`icon_width + item_icon_space`); đặt `icon_width = 0` + `item_icon_space = 0` để lấy lại 36px đó
  khi theme không vẽ icon.
- `GRUB_TERMINAL_OUTPUT` phải unset để theme render.

## Kiểm chứng

Đã boot thật trên VM (QEMU/libvirt, `-vga std`, 1920x1080) với **10 mục menu** (Arch, Advanced options,
Windows 11, Ubuntu, SystemRescue, Fedora, Debian, openSUSE, Void, NixOS): render 70k+ màu, cả 10 dòng
hiển thị đồng thời (pitch 62, thanh chọn cao 58px, font 26), không đè viền trên / dải chấm / logo /
hoa văn dưới (0 px mỗi vùng), scroll đúng (↓ giữ khối cố định, entry cuối dừng ở hàng đáy y 742…799),
label đếm ngược hiển thị. Ảnh: `assets/grub-ciel-vm-10os-font26.png`.
