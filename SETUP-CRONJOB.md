# Pemicu 1 menit melalui cron-job.org

Workflow `.github/workflows/create-vm.yml` menerima `workflow_dispatch` dan melakukan satu percobaan membuat `marketplace-a1` untuk setiap pemicu. cron-job.org memanggil GitHub API setiap menit. GitHub dapat mengantre run, sehingga waktu percobaan tidak dijamin tepat pada detik yang sama setiap menit.

## 1. Buat token GitHub khusus

Di GitHub, buka **Settings > Developer settings > Personal access tokens > Fine-grained tokens > Generate new token**. Pilih pemilik `hbbamdn`, akses **Only select repositories** ke `oci-free-arm-instance`, dan izin repository **Actions: Read and write**. Pilih masa berlaku token yang sesuai. Salin token sekali ke tempat aman. Jangan simpan token di repo, Discord, atau chat.

Dokumentasi izin: https://docs.github.com/en/rest/actions/workflows#create-a-workflow-dispatch-event

## 2. Buat job di cron-job.org

Buat akun di https://cron-job.org/ lalu buat job baru dengan pengaturan:

| Pengaturan | Nilai |
|---|---|
| Title | `OCI A1 Batam` |
| URL | `https://api.github.com/repos/hbbamdn/oci-free-arm-instance/actions/workflows/create-vm.yml/dispatches` |
| Method | `POST` |
| Schedule | Every minute |
| Header `Authorization` | `Bearer <token GitHub fine-grained Anda>` |
| Header `Accept` | `application/vnd.github+json` |
| Header `Content-Type` | `application/json` |
| Request body | `{"ref":"main"}` |

Simpan job dalam keadaan **Disabled** dahulu. Pastikan workflow versi cron-job.org sudah ada pada branch `main` sebelum mengaktifkan job. Jangan menaruh token pada URL; masukkan hanya ke header `Authorization`.

cron-job.org mendukung interval setiap menit, request `POST`, header, dan body khusus: https://cron-job.org/en/faq/

## 3. Siapkan penghentian otomatis

Setelah job tersimpan, buat API key cron-job.org pada pengaturan akun dan catat ID job tersebut. Di GitHub fork, buka **Settings > Secrets and variables > Actions** lalu buat dua repository secrets:

| Secret | Isi |
|---|---|
| `CRONJOB_API_KEY` | API key dari akun cron-job.org |
| `CRONJOB_JOB_ID` | ID numerik job `OCI A1 Batam` |

Setelah VM berhasil dibuat atau ditemukan, workflow mencoba mematikan job cron-job.org melalui API, lalu mematikan workflow GitHub. Jika dua secret ini tidak diisi, pematian job cron-job.org harus dilakukan manual. Pastikan keduanya benar sebelum mengaktifkan jadwal setiap menit.

Dokumentasi API: https://docs.cron-job.org/rest-api.html#updating-a-cron-job

## 4. Uji dan aktifkan

1. Pastikan workflow `OCI A1 Batam Capacity Catcher` berstatus **Active** di GitHub Actions.
2. Di cron-job.org, gunakan **Test run** untuk job tersebut. Respons `204` dari GitHub berarti pemicu diterima, bukan berarti VM sudah dibuat.
3. Periksa run baru di https://github.com/hbbamdn/oci-free-arm-instance/actions. Log `OutOfHostCapacity` berarti OCI belum menyediakan host; error lain perlu diperbaiki sebelum mengaktifkan jadwal.
4. Aktifkan job cron-job.org. Periksa bahwa hanya satu job eksternal yang memicu workflow ini.
5. Setelah ada notifikasi Discord bahwa VM berhasil dibuat, verifikasi instance di OCI Console. Pastikan job cron-job.org **Disabled** dan workflow GitHub **Disabled**. Bila salah satunya masih aktif, matikan manual.

Token GitHub yang kadaluwarsa atau dicabut membuat pemicu gagal. Perbarui header `Authorization` di cron-job.org bila token diganti. Pembatasan API OCI dapat menunda percobaan berikutnya.
