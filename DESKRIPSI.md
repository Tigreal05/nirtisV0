# NIRTIS (Nirvana Tiny Server)

**Ringkasnya:** OS server mini tanpa GUI buat STB HG680P. Semua kendali dan akses lewat web, berupa terminal CLI di browser. Nggak ada tampilan HDMI sama sekali, dan itu memang disengaja.

## Hardware target
- **SoC:** Amlogic S905X (4× Cortex-A53, 1,5 GHz), board P212, DTB `meson-gxl-s905x-p212.dtb`
- **Memori:** RAM 2 GB, eMMC 8 GB, plus SD card
- **Jaringan:** internet lewat dongle USB-LAN DM9601 (LAN internal juga didukung). WiFi RTL8189FTV via SDIO, tapi drivernya out-of-tree
- **Konsol debug:** UART `ttyAML0` di 115200

## Arsitektur
- **Kernel:** Linux 6.6.54 arm64, dikompilasi sendiri dari `defconfig` yang dirampingkan.
- **Userland:** Alpine Linux 3.24 (minirootfs aarch64, musl, OpenRC).
- **Cara jalan:** rootfs dibungkus jadi initramfs terkompresi dan jalan di RAM.
- **Penyimpanan:** SD card dipakai buat database, file web, dan swap. Memori pakai zram/zswap dulu supaya SD nggak cepat aus.
- **Antarmuka:** terminal web (ttyd atau gotty). Ini sama dengan shell root lewat browser, jadi wajib ada autentikasi plus HTTPS, atau taruh di belakang VPN/tunnel.

## Status kernel (sudah jadi)
- `Image` ±16 MB (turun dari 33 MB di build pertama), modul 40 buah dengan total 2,7 MB, DTB 29 KB.
- **Dimatikan:** DRM/grafis, FB, sound, media, PCI, KVM, Bluetooth, HID, CAN, NFS, dan sisa SoC lain.
- **Dipertahankan built-in:** konsol serial, MMC/SD/eMMC, jalur USB (dwc3, xhci, DM9601), ethernet, PWM (clock 32 kHz buat WiFi), watchdog, swap, dan IPv6.
- **Modul:** nftables/iptables, zram, cfg80211/mac80211.
- Script pembuatnya: `nirtis-config-v2.sh`.

## Target ukuran
Estimasi kasar: `Image` ~16 MB + initramfs xz ~6-8 MB + DTB, jadi sekitar **22-25 MB**. Ini mepet terhadap target 25 MB kalau dihitung mentah, tapi masih bisa dipangkas.

## Yang belum dikerjakan
1. Build driver WiFi `rtl8189fs` sebagai modul terpisah.
2. Susun initramfs Alpine lewat `chroot` + qemu (PC lo sudah siap), lengkap dengan OpenRC, terminal web, dan modul kernel.
3. Entri boot. Ini menunggu isi partisi `BOOT_EMMC` (`mmcblk1p1`) dari STB.
4. Tes boot dari SD/USB dulu, dengan UART terpasang. Jangan timpa Armbian di eMMC sebelum tes lolos.
5. Siapkan partisi data dan swap di SD, plus autentikasi dan HTTPS buat terminal web.

## Risiko yang perlu dijaga
- **Jalur USB-LAN adalah satu-satunya akses.** Kalau ada yang salah, board nggak muncul di jaringan, dan HDMI juga sudah mati. UART jadi penyelamat.
- **eMMC bawaan sudah 99% penuh** (sisa ±58 MB). Pastikan backup STB lolos `gzip -t` sebelum sentuh apa pun.
- **Driver rtl8189fs belum pasti cocok dengan 6.6.** Kita baru tahu setelah dicoba build.
- **Swap dan database di SD** bikin kartu cepat aus. Pakai kartu high endurance dan partisi terpisah.

