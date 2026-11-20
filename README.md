# GRUB Theme — Ciel Mocha

GRUB2 gfxmenu theme cho VM Arch Linux (virt-manager, UEFI): nền **Ciel** (nhân vật + khung HUD),
bảng màu **Catppuccin Mocha**, font **BigBlueTerm437 Nerd Font Regular 28**.

## Preview

<img src="assets/grub-ciel-vm-final6.png" alt="GRUB boot screen" width="720">

## Cấu trúc

```
src/catppuccin-mocha-grub-theme/
├── theme.txt                   # cấu hình theme
├── background.png              # 1920x1080, PNG24 8-bit, non-interlaced (nhân vật + khung HUD)
├── ciel_bg_grub_edited_2.jpg   # bản export gốc từ Figma (1922x1083 — KHÔNG dùng trực tiếp)
├── item_c.png                  # thanh option chưa chọn  rgba(30,30,46,0.65)
├── select_c.png                # thanh option đang chọn  rgba(180,190,254,0.30)
├── logo_ciel.png               # 224x224 PNG32, lavender #B4BEFE (hiện KHÔNG dùng trong theme.txt)
├── bigblue28.pf2               # font BigBlueTerm437 Nerd Font Regular 28
├── font.pf2                    # font cũ của upstream (không dùng)
└── icons/                      # icon hệ điều hành cho menu entry
```

## Cài đặt

1. Copy theme vào `/boot/grub/themes/` (thay `TEN_THEME` bằng tên thư mục bạn muốn):

   ```sh
   sudo cp -r src/catppuccin-mocha-grub-theme /boot/grub/themes/TEN_THEME/
   sudo cp src/catppuccin-mocha-grub-theme/bigblue28.pf2 /boot/grub/fonts/
   ```

2. Sửa `/etc/default/grub`:

   ```sh
   GRUB_THEME="/boot/grub/themes/TEN_THEME/theme.txt"
   GRUB_FONTS="bigblue28"
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
| `desktop-image` | `background.png` — nướng sẵn nhân vật + khung HUD, `stretch` |
| `boot_menu` | `left 50%`, `top 29%`, `width 40%`, `height 30%` |
| item | `item_height 80`, `item_spacing 24`, `item_padding 24`, `scrollbar false` |
| font | `BigBlueTerm437 Nerd Font Regular 28` cho cả `item_font`, `selected_item_font`, `terminal-font`, label |
| màu chữ | `#cdd6f4` (Catppuccin Mocha text) |
| nền thanh | `item_pixmap_style = "item"` / `selected_item_pixmap_style = "select"` (khớp `item_c.png`, `select_c.png`) |
| `label` | đồng hồ đếm ngược ở góc dưới-trái (`top 88%`, `left 3%`) |

Mép phải thanh option (90%) nằm trong đường viền HUD (≈93.1% ở ảnh nền) — không đè viền.

## Ghi chú kỹ thuật (đã kiểm chứng trên GRUB 2.16)

- **`grub-mkfont` không có tuỳ chọn `--info`.** Lấy tên nội bộ của file `.pf2` bằng:

  ```sh
  grub-mkfont -s 28 -o bigblue28.pf2 /usr/share/fonts/BigBlueTerm/BigBlueTerm437NerdFont-Regular.ttf
  strings bigblue28.pf2 | grep -A1 PFF2NAME    # -> BigBlueTerm437 Nerd Font Regular 28
  ```

  Tên trong `theme.txt` phải **trùng chính xác** (kể cả cỡ chữ). Bản `...NerdFontMono...ttf` sẽ cho tên có chữ `Mono`.
- **Phần trăm phải là số nguyên** — `61.8%` làm hỏng toàn bộ theme (rơi về menu text-mode); phép tính kiểu `68%-112` không được hỗ trợ, phải tự quy đổi ra `%` nguyên.
- **Không dùng `+ image` đè lên vùng menu**: GRUB vẽ image lên trên menu → chữ menu bị che. Khung HUD phải nướng sẵn vào ảnh nền.
- `item_pixmap_style`/`selected_item_pixmap_style` chấp nhận cả dạng ngắn `"item"`/`"select"` **và** dạng mẫu `"item_*.png"`; chỉ cần file `_c.png` (tâm), các lát cạnh/góc là tuỳ chọn.
- Ảnh nền phải **đúng 1920x1080, PNG24 8-bit, non-interlaced** (JPEG 1922x1083 gây xé/lệch ảnh khi GRUB stretch):
  ```sh
  magick <nguồn> -crop 1920x1080+1+1 +repage -define png:format=png24 -depth 8 -interlace none -strip PNG24:background.png
  ```

## Kiểm chứng

Đã boot thật trên VM (QEMU/libvirt, `-vga std` 16 MiB, 1920x1080): theme render 70k+ màu, thanh chọn
cao 80px, chữ font 28 một dòng không bị cắt, label đếm ngược hiển thị. Ảnh: `assets/grub-ciel-vm-final6.png`.
