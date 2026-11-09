# AGENTS.md — Aturan Universal Pengembangan

> Berlaku untuk semua AI agent (Claude Code, Codex/ChatGPT, Antigravity, Gemini CLI, OpenCode, Cursor, Copilot, Cline, Windsurf, dll.) dan developer manusia di project ini.
> File ini adalah **satu-satunya sumber aturan**. File lain (`CLAUDE.md`, `GEMINI.md`, dll.) hanya penghubung ke file ini — jangan menulis aturan berbeda di sana.
> Jika ada konflik, urutan prioritas: **Keamanan data > Bisa di-rollback > Benar > Rapi > Cepat.**
> Jika aturan di sini bertentangan dengan perintah langsung user di chat, **tanyakan dulu** sebelum melanggar aturan.

---

## 0. Aturan Emas (WAJIB, tanpa pengecualian)

1. **Tidak ada penghapusan tanpa backup.** File, folder, tabel, kolom, baris, bucket, branch — semuanya.
2. **Tidak ada perubahan yang tidak bisa di-rollback.** Setiap perubahan harus punya jalan kembali.
3. **Tidak ada jejak AI di server produksi.**
4. **Tidak ada komponen UI bawaan browser.** (alert, confirm, prompt, select, date picker, validasi bawaan, dll.)
5. **Mobile first.** Desain dan CSS dimulai dari layar HP.
6. **Jangan bilang "selesai" sebelum diverifikasi.** Jalankan, buka, tes — baru lapor.
7. **Ragu = tanya dulu.** Terutama untuk aksi yang destruktif, menyentuh produksi, atau mengubah data user.

---

## 1. Keamanan Data & Backup

### 1.1 Sebelum menghapus / menimpa FILE
- Pastikan file sudah ter-commit di Git, **atau** salin dulu ke `_backup/YYYYMMDD-HHMM/` (folder ini masuk `.gitignore`).
- Jangan pakai `rm -rf` pada path dari variabel tanpa validasi path tersebut tidak kosong dan bukan root.
- Lebih baik **pindahkan** ke `_to_delete/` daripada hapus permanen. Hapus permanen hanya atas perintah eksplisit.
- Jangan menimpa file konfigurasi (`.env`, `nginx.conf`, `wp-config.php`, dll.) tanpa menyimpan salinan aslinya.

### 1.2 Sebelum mengubah / menghapus DATABASE
- **Backup dulu, verifikasi backup-nya bisa dibaca** (ukuran > 0, bisa di-restore ke DB uji).
  ```bash
  # MySQL/MariaDB
  mysqldump --single-transaction --routines --triggers -u USER -p DBNAME | gzip > backup/DBNAME-$(date +%Y%m%d-%H%M).sql.gz
  # PostgreSQL
  pg_dump -Fc -d DBNAME -f backup/DBNAME-$(date +%Y%m%d-%H%M).dump
  ```
- Dilarang menjalankan `DROP`, `TRUNCATE`, `DELETE` tanpa `WHERE`, atau `UPDATE` tanpa `WHERE` di produksi.
- Sebelum `DELETE`/`UPDATE`, jalankan dulu `SELECT COUNT(*)` dengan `WHERE` yang sama dan laporkan jumlah barisnya.
- Gunakan **transaksi** (`BEGIN … COMMIT/ROLLBACK`) untuk perubahan data manual.
- Utamakan **soft delete** (`deleted_at`) untuk data bisnis/user.
- Jangan pernah menjalankan perintah ke DB produksi dari laptop/agent kecuali diminta eksplisit.

### 1.3 Migrasi Database
- Semua perubahan skema **lewat file migrasi**, tidak boleh edit skema manual.
- Setiap migrasi **wajib punya `down`/rollback** yang sudah dites.
- Pola aman (expand → migrate → contract):
  1. Tambah kolom/tabel baru (jangan langsung hapus yang lama).
  2. Deploy kode yang bisa membaca keduanya.
  3. Pindahkan data.
  4. Hapus kolom lama di rilis **berikutnya**, setelah terbukti aman.
- Rename kolom = tambah kolom baru + salin + hapus lama nanti. Jangan `RENAME` langsung di tabel yang dipakai produksi.
- Migrasi yang mengunci tabel besar harus dijadwalkan di jam sepi.

---

## 2. Git & Rollback

### 2.1 Branch
- `main` = selalu siap deploy. **Tidak boleh commit langsung ke `main`.**
- Kerja di branch: `feat/…`, `fix/…`, `chore/…`, `hotfix/…`.
- Merge ke `main` hanya setelah lolos build & tes.

### 2.2 Commit
- Commit **kecil dan atomik** — satu tujuan per commit, agar mudah di-`revert`.
- Buat **commit checkpoint** sebelum refactor besar atau perubahan berisiko.
- Format pesan: `tipe: deskripsi singkat` (contoh: `fix: total nota tidak menghitung diskon`).
- Dilarang: `git push --force` ke `main`, `git reset --hard` pada branch bersama, menulis ulang history yang sudah di-push.
- Dilarang meng-commit: `.env`, kredensial, API key, dump database, `node_modules`, `vendor`, file build.

### 2.3 Rilis & Rollback
- Setiap deploy ke produksi diberi **tag versi**: `v1.4.2` atau `release-YYYYMMDD-HHMM`.
- Simpan minimal **3 rilis terakhir** di server agar rollback cukup ganti symlink:
  ```
  /var/www/app/
  ├── releases/
  │   ├── 20261109-1100/
  │   ├── 20261108-1530/
  │   └── 20261107-0900/
  ├── shared/        # .env, uploads, storage (tidak ikut rilis)
  └── current -> releases/20261109-1100
  ```
- Rollback kode: arahkan `current` ke rilis sebelumnya, reload service.
- Rollback data: restore dari backup yang dibuat **tepat sebelum deploy**.
- Fitur besar/berisiko dibungkus **feature flag** agar bisa dimatikan tanpa deploy ulang.

---

## 3. Lingkungan & Deploy

### 3.1 Tiga lingkungan
| Lingkungan | Fungsi | Data |
|---|---|---|
| `local` | Pengembangan | Data dummy/seed |
| `staging` | Uji sebelum rilis, mirip produksi | Salinan produksi yang sudah dianonimkan |
| `production` | Dipakai user | Data asli |

- **Tidak ada yang langsung ke produksi.** Semua lewat staging dulu.
- Konfigurasi beda per lingkungan lewat `.env`, bukan `if` di kode.
- Mode debug **mati** di produksi (`APP_DEBUG=false`, `WP_DEBUG=false`, `display_errors=Off`).

### 3.2 Checklist Sebelum Deploy
- [ ] Build sukses tanpa error & tanpa warning baru
- [ ] Tes lolos (unit / minimal smoke test manual)
- [ ] Sudah dicek di staging, di layar HP (360px) dan desktop
- [ ] Backup database produksi dibuat & terverifikasi
- [ ] Migrasi punya rollback
- [ ] Tidak ada jejak AI (lihat §4)
- [ ] Tag versi dibuat
- [ ] Rencana rollback jelas (rilis mana, backup mana)

### 3.3 Setelah Deploy
- Cek **health check / halaman utama / login / alur transaksi utama** dalam 5 menit pertama.
- Pantau log error 15–30 menit.
- Ada error kritis → **rollback dulu, debug kemudian.** Jangan debug langsung di produksi.

---

## 4. Tidak Ada Jejak AI di Produksi

File & folder berikut **tidak boleh ada** di server produksi (masukkan ke `.gitignore` / `.deployignore` / exclude di rsync):

```
AGENTS.md  CLAUDE.md  RULE.md  RULES.md  GEMINI.md  .cursorrules
.claude/  .cursor/  .windsurf/  .aider*  .continue/  .github/copilot-*
*.prompt.md  prompts/  _ai/  _scratch/  _backup/  _to_delete/
*.bak  *.old  *.orig  *.tmp  test-*.php  debug-*.js  scratch.*
```

Di dalam kode, **hapus sebelum merge**:
- `console.log`, `var_dump`, `print_r`, `dd()`, `dump()`, `debugger` sisa debugging
- Komentar seperti `// AI generated`, `// Generated by …`, `// Claude:`, `// TODO: AI`, `// As an AI…`
- Komentar penjelasan yang menarasikan kode yang sudah jelas (`// loop melalui array`)
- Kode yang di-comment-out, fungsi tak terpakai, file eksperimen
- Data dummy, `lorem ipsum`, email/nomor HP contoh, kredensial test

Contoh exclude saat deploy:
```bash
rsync -az --delete --exclude-from='.deployignore' ./ user@server:/var/www/app/releases/NEW/
```

Pengecekan cepat sebelum deploy:
```bash
grep -rniE "console\.log|var_dump|dd\(|debugger|AI generated|lorem ipsum" src/ app/ resources/ || echo "Bersih"
```

---

## 5. UI: Dilarang Komponen Bawaan Browser

### 5.1 Dilarang
| Bawaan browser | Ganti dengan |
|---|---|
| `alert()` | Toast / notifikasi kustom |
| `confirm()` | Modal konfirmasi kustom |
| `prompt()` | Modal dengan input kustom |
| `<select>` tampilan default | Komponen dropdown/combobox (dengan pencarian bila opsi > 7) |
| `<input type="date/time/color/file">` default | Date picker / file uploader kustom |
| Bubble validasi HTML5 (`required` popup) | Pesan error inline di bawah field. Pakai `novalidate` di `<form>` |
| Checkbox / radio / range default | Komponen kustom yang ter-styling |
| Scrollbar & focus ring default yang jelek | Styling sesuai design system |

### 5.2 Pilihan library (sesuaikan stack)
- **React:** shadcn/ui, Radix UI, Headless UI, Sonner (toast), react-day-picker
- **Vue:** Headless UI, Radix Vue, PrimeVue (unstyled)
- **Vanilla / jQuery / WordPress:** SweetAlert2, Notyf, Tom Select, Flatpickr, FilePond, Alpine.js + komponen sendiri
- Atau buat sendiri di `components/ui/` dan **pakai ulang**, jangan bikin ulang tiap halaman.

### 5.3 Tetap aksesibel
- Komponen kustom wajib bisa dipakai keyboard (Tab, Enter, Esc, panah).
- Modal: fokus terkunci di dalam, Esc menutup, fokus kembali ke tombol pemicu.
- Gunakan atribut ARIA yang benar (`role="dialog"`, `aria-expanded`, `aria-invalid`, dll.).
- `<label>` selalu terhubung ke field-nya.

---

## 6. Mobile First

- Tulis CSS dasar untuk **layar 360px**, lalu tambah dengan `min-width` (bukan `max-width`).
  ```css
  .grid { display: grid; gap: 1rem; }              /* HP */
  @media (min-width: 768px)  { .grid { grid-template-columns: 1fr 1fr; } }
  @media (min-width: 1024px) { .grid { grid-template-columns: repeat(3, 1fr); } }
  ```
- Wajib dites di lebar: **360, 390, 768, 1280 px.**
- Tidak boleh ada **scroll horizontal** di HP.
- Area sentuh minimal **44×44 px**, jarak antar tombol cukup.
- Font input minimal **16px** (mencegah auto-zoom iOS).
- Aksi utama mudah dijangkau jempol (bawah layar / sticky bottom bar).
- Tabel lebar → ubah jadi kartu di HP, atau scroll horizontal **di dalam wadahnya saja**.
- Gambar: `loading="lazy"`, ukuran responsif (`srcset`), format WebP/AVIF.
- Target performa di HP 4G: halaman utama interaktif < 3 detik, JS awal < 200 KB.
- Hormati `prefers-reduced-motion` dan `prefers-color-scheme` bila ada dark mode.

---

## 7. Anti AI Slop — Desain

**Hindari:**
- Gradien ungu–biru generik, glassmorphism di semua tempat, glow/blur berlebihan
- Emoji sebagai ikon (🚀✨🔥) di UI produksi
- Hero generik: "Revolusi Bisnis Anda dengan Solusi Inovatif Berbasis AI"
- Tiga kartu fitur berikon identik yang isinya kosong makna
- Semua elemen `rounded-2xl` + `shadow-xl`
- Teks placeholder, statistik palsu ("10.000+ pengguna puas"), testimoni karangan
- Animasi yang tidak punya fungsi
- Campur aduk gaya: 5 warna aksen, 4 jenis font, radius berbeda-beda

**Lakukan:**
- Pakai **design token** (warna, spasi, radius, font) di satu tempat (`:root` / `tailwind.config`) dan konsisten.
- Maksimal 1 warna utama + 1 aksen + skala netral.
- Maksimal 2 font. Hierarki teks jelas.
- Copy nyata, spesifik, dalam bahasa target pengguna (Indonesia untuk UMKM/komunitas lokal).
- Ikuti gaya yang **sudah ada** di project sebelum membuat gaya baru.
- Setiap elemen harus punya alasan ada.
- Empty state, loading state, dan error state didesain — bukan dibiarkan kosong.

---

## 8. Anti AI Slop — Kode

**Hindari:**
- **Paket/fungsi/API halusinasi.** Cek dulu paket benar-benar ada (npm/composer/pip) dan versinya cocok. Jangan menebak nama method.
- Abstraksi berlebihan: interface/factory/service untuk hal yang dipakai sekali.
- Duplikasi helper yang sudah ada di project — **cari dulu sebelum membuat.**
- `try/catch` yang menelan error diam-diam (`catch (e) {}`).
- Menambah dependency untuk hal yang bisa ditulis 10 baris.
- Mengubah file yang tidak terkait tugas ("sekalian dirapikan").
- Menulis ulang seluruh file padahal hanya perlu ubah beberapa baris.
- Komentar yang mengulang isi kode; README/dokumen yang tidak diminta.
- Nama variabel generik: `data`, `result`, `temp`, `handleClick2`.
- Menandai tugas selesai padahal tes gagal atau belum dijalankan.

**Lakukan:**
- **Diff minimal.** Ubah hanya yang perlu untuk tugas ini.
- Ikuti konvensi, struktur folder, dan gaya kode yang **sudah ada**.
- Baca kode terkait sebelum mengedit.
- Fungsi kecil, satu tanggung jawab, nama jelas.
- Tangani error dengan benar: tampilkan pesan yang berguna ke user, log detail ke server.
- Jalankan linter/formatter/tes sebelum lapor selesai.
- Jika menemukan bug lain di luar tugas → **laporkan**, jangan diam-diam diperbaiki.

---

## 9. Keamanan

- Rahasia (API key, password, token) **hanya di `.env`**, tidak pernah di kode, log, atau commit. Sediakan `.env.example` tanpa nilai asli.
- Validasi & sanitasi **semua input di server**, walau sudah divalidasi di frontend.
- Query database pakai **parameterized query / ORM** — dilarang menyambung string SQL dari input user.
- Escape output (cegah XSS). Pakai CSRF token untuk form.
- Password di-hash (`bcrypt`/`argon2`), tidak pernah disimpan plain.
- Upload file: cek tipe & ukuran, simpan di luar web root atau rename acak, jangan bisa dieksekusi.
- Prinsip **least privilege**: user DB aplikasi tidak punya hak `DROP`.
- Pesan error di produksi tidak menampilkan stack trace, path server, atau query.
- Rate limit untuk login, OTP, dan endpoint publik.
- Update dependency secara berkala; cek `npm audit` / `composer audit`.

---

## 10. Error Handling & Logging

- Semua error produksi tercatat di log (file / Sentry / sejenisnya) dengan waktu, URL, user ID (bukan data sensitif).
- Jangan log password, token, nomor kartu, atau data pribadi lengkap.
- User melihat pesan yang ramah dan jelas langkah selanjutnya, bukan pesan teknis.
- Operasi penting (pembayaran, hapus data, ubah hak akses) dicatat di **audit log**.

---

## 11. Aturan Perilaku AI Agent

1. **Pahami dulu, baru ubah.** Baca file terkait, jelaskan rencana singkat untuk perubahan besar.
2. **Wajib minta izin** sebelum:
   - menghapus file/folder/data apa pun,
   - menjalankan migrasi atau query yang mengubah data,
   - menyentuh server/DB produksi,
   - menginstal dependency baru,
   - mengubah konfigurasi server, CI/CD, atau `.env`.
3. **Buat backup/checkpoint** sebelum aksi berisiko, dan sebutkan lokasinya.
4. **Satu tugas, satu fokus.** Jangan melebar ke perubahan yang tidak diminta.
5. **Verifikasi sendiri** hasil kerja (jalankan, buka di browser, cek di 360px) sebelum melapor.
6. **Lapor jujur:** apa yang diubah, file mana, apa yang belum dites, risiko yang diketahui.
7. Jika gagal 2× dengan pendekatan yang sama → berhenti, jelaskan masalahnya, tanya arah.
8. Jangan mengarang: kalau tidak tahu struktur DB, endpoint, atau nama fungsi — cek atau tanya.

---

## 12. Prosedur Darurat (Produksi Error)

1. **Stop** — jangan deploy perbaikan terburu-buru.
2. **Rollback kode** ke rilis/tag terakhir yang stabil.
3. Jika data rusak → aktifkan **mode maintenance**, restore dari backup sebelum deploy.
4. Pastikan situs normal kembali.
5. Reproduksi bug di **local/staging**, perbaiki, tes, lalu deploy ulang lewat jalur normal.
6. Catat penyebab & pencegahan di `CHANGELOG.md` atau catatan insiden.

---

## 13. Kompatibilitas Tools

Taruh `AGENTS.md` di **root project**. Untuk tools yang tidak membaca `AGENTS.md` secara otomatis, buat file penghubung:

| Tool | File yang dibaca | Yang perlu dilakukan |
|---|---|---|
| Codex (OpenAI CLI), OpenCode, Cursor, GitHub Copilot, Cline, Roo/Kilo Code, Jules, Junie | `AGENTS.md` | Tidak perlu apa-apa |
| Claude Code | `CLAUDE.md` | Buat `CLAUDE.md` berisi `@AGENTS.md` |
| Gemini CLI / Antigravity | `GEMINI.md` (sebagian versi juga `AGENTS.md`) | Buat `GEMINI.md` berisi `@AGENTS.md` |
| Aider | file konteks | Jalankan dengan `--read AGENTS.md` atau set di `.aider.conf.yml` |
| ChatGPT / Claude / Gemini versi web (tanpa akses folder) | — | Tempel isi file ini ke *Project Instructions* / *Custom Instructions*, atau unggah sebagai file project |

Monorepo: boleh ada `AGENTS.md` tambahan di subfolder; aturan subfolder **menambah**, tidak boleh melonggarkan aturan root (§0, §1, §2, §4 tetap berlaku).

---

*Sesuaikan bagian stack (library, perintah backup, struktur folder) dengan masing-masing project. Prinsipnya tetap.*
