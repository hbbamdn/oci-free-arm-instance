# Setup VM marketplace di OCI Batam

Konfigurasi lokal ada di `.github/workflows/create-vm.yml`: A1.Flex, 2 OCPU, 4 GB RAM, boot volume 50 GB, Ubuntu 24.04 ARM64, region `ap-batam-1`, nama `marketplace-a1`. GitHub memulai run setiap 5 menit; tiap run mencoba hingga 5 kali dengan jeda 60 detik saat kapasitas penuh. Jadwal GitHub bisa terlambat, sehingga jarak antarpercobaan tidak dijamin tepat 1 menit. Workflow mencoba menonaktifkan diri setelah VM dibuat atau ditemukan. Kapasitas host tidak dijamin.

## 1. Periksa jatah OCI

Di OCI Console, periksa semua instance A1, boot volume, block volume, dan backup di home region Batam. Jatah Always Free A1 saat ini setara 2 OCPU/12 GB; penyimpanan gabungan 200 GB. Konfigurasi ini memakai seluruh jatah OCPU A1, 4 GB dari jatah RAM A1, dan 50 GB penyimpanan. Jangan mulai jika jatah sudah terpakai atau ukuran ini akan menimbulkan biaya pada akun Anda.

## 2. Fork dan salin workflow

1. Fork `https://github.com/NarahaYuu/oci-free-arm-instance` ke akun GitHub Anda.
2. Pada fork, buka `.github/workflows/create-vm.yml`, klik Edit, lalu ganti isinya dengan file lokal ini dan commit ke branch default fork. Jangan push ke repo sumber.
3. Pastikan bagian atas file memuat `schedule` dan `workflow_dispatch`, serta nilai `INSTANCE_NAME=marketplace-a1`, `OCPUS=2`, `MEMORY_GB=4`, `BOOT_VOLUME_GB=50`.

## 3. Isi GitHub Actions Secrets di fork

Buka Settings > Secrets and variables > Actions > New repository secret. Isi nama berikut dari akun OCI Anda:

- `OCI_COMPARTMENT_ID`: compartment tujuan VM.
- `OCI_SUBNET_ID`: subnet Batam yang sesuai untuk VM. Workflow meminta public IP.
- `OCI_CLI_USER`: OCID user API.
- `OCI_CLI_TENANCY`: OCID tenancy.
- `OCI_CLI_FINGERPRINT`: fingerprint API key.
- `OCI_CLI_KEY_CONTENT`: seluruh isi private API key OCI. Jangan masukkan ke file repo atau chat.
- `OCI_CLI_PASSPHRASE`: passphrase API key jika key terenkripsi; jika tidak, biarkan kosong.
- `SSH_PUBLIC_KEY`: public key SSH untuk login ke VM. Simpan private SSH key hanya di perangkat Anda.
- `DISCORD_WEBHOOK_URL`: webhook Discord bila ingin notifikasi. Jika kosong, langkah notifikasi dilewati dan workflow tetap dapat membuat VM.

Gunakan user API OCI dengan izin seminimal mungkin pada compartment tujuan. Jangan commit credential atau file `.pem`.

## 4. Jalankan dan verifikasi

1. Di tab Actions fork, aktifkan workflow terjadwal bila GitHub menonaktifkannya setelah fork.
2. Jalankan `OCI A1 Batam Capacity Catcher` sekali lewat Run workflow untuk memeriksa autentikasi dan konfigurasi.
3. Jika hasilnya `OutOfHostCapacity`, biarkan jadwal mencoba lagi. Untuk error lain, baca log Actions dan perbaiki penyebabnya.
4. Setelah berhasil, cek instance `marketplace-a1` di OCI Console, ukuran dan boot volume, lalu pastikan workflow sudah Disabled di GitHub Actions. Penonaktifan otomatis dapat gagal.
5. Atur firewall/security list agar hanya port yang diperlukan terbuka; aktifkan HTTPS dan backup database sebelum menerima transaksi.

Dokumentasi: https://docs.oracle.com/en-us/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm
