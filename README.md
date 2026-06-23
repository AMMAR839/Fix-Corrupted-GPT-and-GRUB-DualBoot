## 🛠️ Langkah-Langkah Pemulihan

### Fase 1: Memulihkan Tabel Partisi Utama (GPT) dari Salinan Cadangan

Karena salinan cadangan (*backup GPT*) masih utuh, kita bisa menggunakannya untuk menimpa tabel utama yang rusak menggunakan tool `gdisk`.

1. Pastikan tidak ada aplikasi partisi (seperti `parted`) yang sedang tertahan di terminal. Jika ada prompt `OK/Cancel?`, ketik `Cancel`.
2. Buka terminal di Live USB dan jalankan `gdisk` untuk drive SSD/HDD yang bermasalah:

```bash
   sudo gdisk /dev/nvme0n1

```

3. `gdisk` akan otomatis mendeteksi kerusakan dan memuat tabel partisi dari backup. Pada prompt `Command (? for help):`, ketik **`w`** (write) untuk menulis perbaikan ke disk, lalu tekan **Enter**.
4. Saat muncul konfirmasi tindakan, ketik **`Y`** dan tekan **Enter**.
5. Beritahu kernel untuk membaca ulang struktur partisi yang baru diperbaiki:

```bash
   sudo partprobe /dev/nvme0n1

```

6. Pastikan partisi sudah muncul kembali dengan memeriksa daftar disk:

```bash
   lsblk

```

*Pastikan `nvme0n1p1` (EFI) dan `nvme0n1p5` (Root Ubuntu) sudah terlihat kembali.*

### Fase 2: Mengaitkan (Mount) Partisi untuk Proses Chroot

Sebelum menginstal ulang GRUB, kita harus masuk ke dalam sistem Linux asli yang terpasang di SSD (*chroot*) dari lingkungan Live USB.

1. Kaitkan partisi Root Ubuntu (misal: `nvme0n1p5`) ke direktori `/mnt`:

```bash
   sudo mount /dev/nvme0n1p5 /mnt

```

2. Buat direktori untuk EFI dan kaitkan partisi EFI (misal: `nvme0n1p1`):

```bash
   sudo mkdir -p /mnt/boot/efi
   sudo mount /dev/nvme0n1p1 /mnt/boot/efi

```

3. Kaitkan sistem virtual kernel agar lingkungan chroot dapat mengenali hardware dan variabel sistem:

```bash
   sudo mount --bind /dev /mnt/dev
   sudo mount --bind /proc /mnt/proc
   sudo mount --bind /sys /mnt/sys
   sudo mount --bind /run /mnt/run

```

### Fase 3: Masuk ke Sistem Asli (Chroot) & Membuka Akses Variabel EFI

1. Masuk ke lingkungan sistem operasi asli di SSD:

```bash
   sudo chroot /mnt

```

*Prompt terminal akan berubah menjadi `root@ubuntu:/#`.*

2. **PENTING:** Jika saat menginstal GRUB muncul peringatan *`grub-install: warning: EFI variables cannot be set on this system`*, itu berarti akses ke NVRAM motherboard terblokir. Buka aksesnya dengan perintah...

```bash
   mount -t efivarfs efivarfs /sys/firmware/efi/efivars

```

### Fase 4: Menginstal Ulang dan Memperbarui GRUB

1. Jalankan instalasi GRUB ke drive utama (bukan ke partisinya):

```bash
   grub-install /dev/nvme0n1

```

*Pastikan outputnya menunjukkan `Installation finished. No error reported.` tanpa peringatan variabel EFI.*

2. Perbarui daftar menu booting GRUB agar mengenali OS Ubuntu dan Windows Boot Manager yang ada di disk:

```bash
   update-grub

```

### Fase 5: Keluar dan Memulai Ulang Sistem

1. Keluar dari lingkungan chroot:

```bash
   exit

```

2. Lakukan restart pada komputer:

```bash
   sudo reboot

```

3. **Segera cabut Flashdisk Live USB** ketika layar laptop mulai mati/gelap.

---

## 📝 Catatan Tambahan

* Panduan ini berasumsi skema partisi menggunakan drive `/dev/nvme0n1` dengan partisi EFI di `/dev/nvme0n1p1` dan partisi root Ubuntu di `/dev/nvme0n1p5`. Sesuaikan penamaan perangkat (`sda`, `sdb`, dll) jika diterapkan pada perangkat keras yang berbeda.
* Proses pemulihan ini aman dan **tidak menghapus data pengguna atau file sistem** pada Windows maupun Ubuntu karena hanya memperbaiki tabel indeks dan bootloader.

```

```
