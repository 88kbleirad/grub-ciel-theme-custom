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
| `boot_menu` | `left 828`, `top 178`, `width 700`, `height 624` |
| item | `item_height 72`, `item_spacing 19`, `item_padding 3`, `scrollbar false` |
| số dòng hiện | 7 dòng; `rows = (H + spacing − 2*padding) / (item_height + spacing)` ⇒ 7 khi `H = 624` (`7*72 + 6*19 + 2*3`) |
| font | `BigBlueTerm437 Nerd Font Regular 24` cho `item_font`/`selected_item_font`; `…28` cho `terminal-font` + label |
| màu chữ | `#cdd6f4` (Catppuccin Mocha text) |
| nền thanh | `item_pixmap_style = "item_*.png"` / `selected_item_pixmap_style = "select_*.png"` (bắt buộc có `*` — GRUB 2.16 trả lỗi pattern thiếu `*` → box rỗng) |
| `label` | đồng hồ đếm ngược ở góc dưới-trái (`top 88%`, `left 3%`) |

Menu nằm từ x 831 đến 1522 (thanh chọn 7 dòng): viền trái HUD ở 786..825, dải chấm/glyph trang trí
848..912 — chữ bắt đầu 867 nên **không đè** dải; avatar bắt đầu ≈1532 (cách 10px — không chạm).
Menu dọc: thanh y 181..252 ... 727..798 (7 hàng, pitch 91). Viền trên HUD kết thúc 178 (cách 3px),
hoa văn dưới bắt đầu 805 (cách 7px). Cả hai biên đều sạch (0px đè).

Cuộn dùng `make_selected_item_visible` của GRUB: khối menu **không di chuyển**, chỉ `first_shown_index`
đổi. Với 10 option, bấm ↓ tới option cuối thì option đó luôn nằm ở hàng đáy (y 727...798, cách hoa văn 7px);
bấm ↑ thì chỉ thanh chọn chạy lên trong khối đứng yên. Viewport chữ 694px — tên dài nhất
"Windows Boot Manager (on /dev/nvme0n1p1)" (640px @24) vừa khít, không cắt.

## Lưu ý GRUB 2.16 (đã kiểm chứng)

- `grub-mkfont` không có `--info`; tên font trong `PF2NAME` phải khớp chính xác (vd `BigBlueTerm437 Nerd Font Regular 24`).
- Pixmap style phải chứa `*` (`item_*.png`); `"item"` không có `*` → `GRUB_ERR_BAD_ARGUMENT`, thanh option không vẽ.
- `height` boot_menu điều khiển số dòng hiển thị: công thức đầy đủ
  `rows = (height + item_spacing − 2*item_padding) / (item_height + item_spacing)`.
- Chữ dài hơn viewport sẽ bị cắt im ở viền phải (`x = 1478`) — không tự wrap. Entry dài nhất thông thường
  ("Advanced options for Arch Linux", 496px @24) vừa khít trong `width 28%` (viewport 506px).
- `GRUB_TERMINAL_OUTPUT` phải unset để theme render.

## Kiểm chứng

Đã boot thật trên VM (QEMU/libvirt, `-vga std` 16 MiB, 1920x1080): theme render 70k+ màu, 5 dòng
menu hiển thị đồng thời, thanh chọn cao 72px, scroll hoạt động (phím ↓ di chuyển thanh chọn),
label đếm ngược hiển thị. Ảnh: `assets/grub-ciel-vm-final6.png`.
