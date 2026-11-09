# Universal AI Agent Rules & Anti-Slop System

Koleksi aturan universal, protokol keamanan, dan pedoman pengembangan untuk **AI Coding Agents** (Google Antigravity, Claude Code, Gemini CLI, OpenAI Codex, Cursor, GitHub Copilot, Cline, Windsurf, Roo Code, dll.) serta pengembang manusia.

Repository ini dirancang sebagai **Single Source of Truth** agar agen AI tidak merusak data produksi, tidak berhalusinasi, tidak menghasilkan desain generik (*AI slop*), dan menjaga kode tetap aman, rapi, serta dapat di-rollback kapan saja.

---

## 📁 Berkas & Struktur Utama

- **[`AGENTS.md`](AGENTS.md)** — Dokumen aturan universal & satu-satunya sumber kebenaran (Aturan Emas, Keamanan Data, Git & Rollback, Anti AI Slop, No Native Browser UI, Mobile First, Error Handling).
- **[`CLAUDE.md`](CLAUDE.md)** — Berkas penghubung otomatis untuk Claude Code (`@AGENTS.md`).
- **[`GEMINI.md`](GEMINI.md)** — Berkas penghubung untuk Gemini CLI / Google Antigravity (`@AGENTS.md`).
- **[`skills/`](skills/)** — Pustaka skill spesialis frontend, visual design, dan animasi tingkat Awwwards (diadaptasi dari [VibeCurb](https://github.com/Yu-369/VibeCurb)).

---

## 🛡️ Prinsip Utama dalam AGENTS.md

1. **Aturan Emas (Golden Rules):** AI wajib menjaga integritas data produksi, tidak pernah menjalankan query perusak tanpa izin, dan selalu membuat snapshot/backup.
2. **Git & Rollback Disiplin:** Setiap perubahan fungsional atau visual harus dalam commit atomic yang dapat di-rollback secara instan.
3. **No Native Browser Select / Alert:** Wajib menggunakan custom component DOM yang elegan, dilarang memakai `<select>` bawaan OS atau `alert()`/`confirm()`.
4. **Mobile First:** Desain harus teruji rapi di viewport HP 360px tanpa horizontal scroll liar.
5. **Sacred vs. Slop (Protokol Bedah Visual):**
   - **SACRED (Haram disentuh):** State (`useState`), effect (`useEffect`), API fetch, event handler logic, routing, dan validasi data tidak boleh dirusak saat memoles tampilan.
   - **SLOP (Upgrade agresif):** CSS, styling tokens, typography scale, padding/margin, dan micro-interactions ditingkatkan ke standar premium.
6. **Pencegahan AI Drift (Drift Rejection):** Menolak secara tegas kemunduran AI ke pola template murahan (gradien ungu-biru generik, glassmorphism berlebihan, transisi `ease` kaku, dan placeholder palsu).

---

## 🎨 Modul Skills Desain (`skills/`)

Diadaptasi dari arsitektur [VibeCurb](https://github.com/Yu-369/VibeCurb) untuk mencegah AI menghasilkan desain web yang membosankan:

| Skill | Kategori | Fungsi Utama |
| :--- | :--- | :--- |
| **[`skills/visual-redesign`](skills/visual-redesign/SKILL.md)** | Redesign | Upgrade estetika kode React/HTML yang kaku menjadi Awwwards-grade **tanpa merusak satu baris pun logika JS/state**. |
| **[`skills/pixel-perfect`](skills/pixel-perfect/SKILL.md)** | Frontend | Replikasi presisi kode dari referensi tangkapan layar/mockup (font, rasio, warna, dan spacing 1:1). |
| **[`skills/awwwards-hero`](skills/awwwards-hero/SKILL.md)** | Frontend | Pembuatan hero section berdampak tinggi dengan tipografi ekspresif dan palet warna ketat. |
| **[`skills/awwwards-sections`](skills/awwwards-sections/SKILL.md)** | Frontend | Membangun section bento grid, pricing table, dan footer interaktif berstandar studio premium. |
| **[`skills/awwwards-motion`](skills/awwwards-motion/SKILL.md)** | Animasi | Koreografi motion dengan fisika (spring), scroll-linked reveals, dan kepatuhan `prefers-reduced-motion`. |
| **[`skills/brandkit-gen`](skills/brandkit-gen/SKILL.md)** | Identitas | Ekstraksi sistem identitas visual, tone, dan palet merek sebelum pembuatan antarmuka. |
| **[`skills/imagegen-frontend`](skills/imagegen-frontend/SKILL.md)** | Grafis | Panduan instruksi pembuatan aset visual dan editorial photography untuk kebutuhan web. |

---

## 🚀 Panduan Penggunaan

### 1. Di Root Project Anda
Cukup salin berkas `AGENTS.md` ke direktori root repository/proyek Anda:
```bash
cp /path/to/MD/AGENTS.md ./AGENTS.md
```

### 2. Penghubung Tools AI
- **Claude Code:** Sertakan berkas `CLAUDE.md` yang merujuk ke `@AGENTS.md`.
- **Gemini CLI / Google Antigravity:** Sertakan berkas `GEMINI.md` yang merujuk ke `@AGENTS.md`.
- **Cursor / Copilot / Cline / Roo Code:** Otomatis membaca `AGENTS.md` di root workspace.
- **Skills:** Salin folder `skills/` ke direktori `.agents/skills/` atau `.cursor/rules/` bila ingin mengaktifkan pipeline skill visual tertentu.

---

## 🤝 Atribusi & Referensi
- Aturan arsitektur agen & database disesuaikan untuk standar pengembangan produksi reliabel.
- Arsitektur anti-slop visual, Sacred vs. Slop, dan pipeline Awwwards diadaptasi dan terinspirasi dari [Yu-369/VibeCurb](https://github.com/Yu-369/VibeCurb).
