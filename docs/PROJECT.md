# Portfolio Website: Project Documentation

> Versi dokumen: 0.1.0 | Status: Draft | Terakhir diperbarui: 2026-10-02
> Pemilik: <nama kamu> | Repo: <link GitHub>

---

## 1. Ringkasan Proyek

### 1.1 Latar Belakang
Portofolio digital untuk mendokumentasikan perjalanan kuliah di Pendidikan Teknik
Informatika: project, prestasi, pengalaman, kekayaan intelektual, dan mahasiswa
bimbingan. Seluruh konten dikelola secara dinamis lewat halaman admin, tanpa
perlu mengubah kode setiap ada pembaruan.

### 1.2 Tujuan
| ID | Tujuan | Indikator Berhasil |
|----|--------|--------------------|
| G1 | Menampilkan seluruh pencapaian dalam satu situs | Semua section tampil dan terisi |
| G2 | Konten dapat dikelola tanpa coding | CRUD semua modul lewat /admin |
| G3 | Biaya operasional nol | Berjalan di free tier |
| G4 | Mudah ditemukan dan dibagikan | Skor Lighthouse >= 90, ada meta SEO dan OG image |

### 1.3 Ruang Lingkup
**In scope:** situs publik, panel admin, autentikasi admin, upload gambar,
form kontak, SEO dasar, deployment.
**Out of scope (v1):** multi-user, komentar, blog, multi-bahasa, analitik kustom.

### 1.4 Asumsi
- Admin hanya satu orang (pemilik).
- Bahasa konten: Indonesia (struktur disiapkan agar mudah ditambah bahasa lain).
- Hosting di Vercel (free tier), database dan storage di Supabase (free tier).
- Desain standar dulu, dipoles setelah fungsi selesai.

---

## 2. Tech Stack

| Layer | Teknologi | Alasan |
|-------|-----------|--------|
| Framework | Next.js (App Router) + TypeScript | SSR/ISR, routing, Server Actions |
| Styling | Tailwind CSS | Cepat dan konsisten |
| Database | Supabase (PostgreSQL) | Gratis, ada RLS |
| Auth | Supabase Auth (email + password) | Terintegrasi dengan RLS |
| Storage | Supabase Storage | Gambar thumbnail, sertifikat, foto |
| Validasi | Zod + React Hook Form | Validasi form di client dan server |
| UI (opsional) | shadcn/ui + lucide-react | Komponen siap pakai, mudah dikustom |
| Deployment | Vercel | Integrasi bagus dengan Next.js |
| Kualitas | ESLint, Prettier, Husky (opsional) | Kode konsisten |

> Catatan free tier: project Supabase gratis bisa otomatis di-pause jika lama
> tidak ada aktivitas. Siapkan cara mengaktifkannya kembali, atau buat
> health-check berkala.

---

## 3. Arsitektur

```
Browser
   |
Next.js (Vercel)
   |-- Public pages (Server Components, ISR/revalidate)
   |-- Admin pages (protected via middleware)
   |-- Server Actions / Route Handlers (CRUD + validasi Zod)
   |
Supabase
   |-- Auth (session via cookie, @supabase/ssr)
   |-- Postgres (tabel + RLS)
   |-- Storage (bucket: media)
```

### 3.1 Prinsip
1. **Public read, admin write.** Pengunjung hanya bisa membaca data
   berstatus `published`; hanya admin yang bisa menulis.
2. **Validasi dua lapis.** Zod di server, RLS di database.
3. **Konten dinamis.** Tidak ada konten hardcode kecuali teks UI.
4. **Server Components dulu.** Client Component hanya untuk interaksi.

### 3.2 Role
| Role | Akses |
|------|-------|
| Visitor | Baca konten published, kirim pesan kontak |
| Admin | CRUD semua modul, lihat pesan, upload media |

---

## 4. Sitemap

### 4.1 Halaman Publik
| Route | Isi |
|-------|-----|
| `/` | Hero, ringkasan, highlight project dan prestasi, CTA |
| `/about` | Bio, foto, kontak, tautan sosial, unduh CV |
| `/projects` | Daftar project + filter kategori dan tag |
| `/projects/[slug]` | Detail project |
| `/achievements` | Daftar prestasi/lomba |
| `/experience` | Asisten dosen, organisasi, kepanitiaan (timeline) |
| `/skills` | Skill dikelompokkan per kategori |
| `/education` | Pendidikan, sertifikasi, pelatihan |
| `/intellectual-property` | HKI dan publikasi artikel |
| `/mentored` | Project mahasiswa bimbingan + testimoni |
| `/contact` | Form kontak dan link sosial |

### 4.2 Halaman Admin
| Route | Fungsi |
|-------|--------|
| `/admin/login` | Login |
| `/admin` | Dashboard: jumlah data per modul, pesan baru |
| `/admin/profile` | Edit data diri dan CV |
| `/admin/projects` | List, tambah, edit, hapus, reorder |
| `/admin/achievements` | CRUD |
| `/admin/experiences` | CRUD |
| `/admin/skills` | CRUD |
| `/admin/education` | CRUD (termasuk sertifikasi) |
| `/admin/ip` | CRUD HKI dan publikasi |
| `/admin/mentored` | CRUD mahasiswa bimbingan dan testimoni |
| `/admin/messages` | Lihat dan tandai pesan kontak |
| `/admin/media` | Kelola file yang diupload |

---

## 5. Kebutuhan Fungsional

Prioritas: **P0** wajib, **P1** penting, **P2** nice-to-have.

### 5.1 Umum
| ID | Kebutuhan | Prioritas |
|----|-----------|-----------|
| FR-01 | Semua modul memiliki CRUD di admin | P0 |
| FR-02 | Status konten: `draft` / `published` | P0 |
| FR-03 | Urutan tampil dapat diatur (`sort_order` atau drag and drop) | P1 |
| FR-04 | Upload gambar ke Supabase Storage dengan preview | P0 |
| FR-05 | Konten `featured` tampil di beranda | P1 |
| FR-06 | Pencarian dan filter di daftar publik | P2 |

### 5.2 Per Modul
| Modul | Kebutuhan Utama |
|-------|-----------------|
| Profile | Satu baris data (singleton): nama, headline, bio, foto, email, sosial, CV |
| Projects | Judul, slug, ringkasan, deskripsi (Markdown), tech stack, peran, link demo/repo, galeri, kategori, tahun |
| Achievements | Nama lomba, penyelenggara, tingkat, peringkat, tanggal, deskripsi, sertifikat/foto |
| Experiences | Tipe (asdos/organisasi/kepanitiaan/magang), posisi, instansi, periode, poin pencapaian |
| Skills | Nama, kategori (technical/soft/language), level, ikon |
| Education | Jenjang, institusi, periode, IPK (opsional); sertifikasi: nama, penerbit, tanggal, link kredensial |
| IP | Tipe (HKI/artikel), judul, nomor/DOI, penerbit/jurnal, tahun, link, dokumen |
| Mentored | Nama mahasiswa, mata kuliah, judul project, deskripsi, teknologi, link, testimoni, izin tampil |
| Messages | Nama, email, pesan, status baca |

---

## 6. Kebutuhan Non-Fungsional

| ID | Kategori | Target |
|----|----------|--------|
| NFR-01 | Performa | Lighthouse Performance >= 90, gambar dioptimasi `next/image` |
| NFR-02 | Keamanan | RLS aktif di semua tabel, tidak ada service key di client |
| NFR-03 | Responsif | Nyaman dari 360px sampai desktop |
| NFR-04 | Aksesibilitas | Lighthouse Accessibility >= 90, alt text pada gambar |
| NFR-05 | SEO | Metadata per halaman, sitemap.xml, robots.txt, OG image |
| NFR-06 | Maintainability | TypeScript strict, struktur folder konsisten |
| NFR-07 | Anti-spam | Honeypot atau rate limit pada form kontak |

---

## 7. Model Data

### 7.1 Daftar Tabel
`profile`, `projects`, `achievements`, `experiences`, `skills`, `education`,
`certifications`, `ip_records`, `mentored_projects`, `testimonials`,
`contact_messages`

### 7.2 Kolom Umum (semua tabel kecuali profile dan contact_messages)
| Kolom | Tipe | Keterangan |
|-------|------|------------|
| id | uuid (PK, default gen_random_uuid()) | |
| status | text (`draft`/`published`) | default `draft` |
| featured | boolean | default false |
| sort_order | int | default 0 |
| created_at / updated_at | timestamptz | default now() |

### 7.3 Contoh Skema (SQL)
```sql
-- Helper: cek apakah user adalah admin
create table admins (
  user_id uuid primary key references auth.users(id) on delete cascade
);

create or replace function is_admin() returns boolean
language sql stable security definer set search_path = public as $$
  select exists (select 1 from admins where user_id = auth.uid());
$$;

-- Contoh tabel: projects
create table projects (
  id uuid primary key default gen_random_uuid(),
  slug text unique not null,
  title text not null,
  summary text,
  description text,                 -- Markdown
  category text,                    -- web, mobile, data, dll.
  tech_stack text[] default '{}',
  role text,
  demo_url text,
  repo_url text,
  thumbnail_url text,
  gallery text[] default '{}',
  year int,
  status text not null default 'draft' check (status in ('draft','published')),
  featured boolean default false,
  sort_order int default 0,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- RLS
alter table projects enable row level security;

create policy "public read published" on projects
  for select using (status = 'published');

create policy "admin full access" on projects
  for all using (is_admin()) with check (is_admin());
```
> Tabel lain mengikuti pola yang sama (kolom umum + kolom spesifik modul + dua
> policy di atas). `contact_messages`: insert terbuka untuk publik, select/update
> hanya admin.

### 7.4 Storage
- Bucket `media` (public read); upload/hapus hanya admin.
- Konvensi path: `{modul}/{id}/{timestamp}-{nama-file}`
- Batas ukuran upload: 2 MB untuk gambar, 5 MB untuk dokumen.

---

## 8. Struktur Folder

```
src/
├─ app/
│  ├─ (public)/
│  │  ├─ page.tsx
│  │  ├─ about/ projects/ achievements/ experience/
│  │  ├─ skills/ education/ intellectual-property/ mentored/ contact/
│  ├─ admin/
│  │  ├─ login/
│  │  ├─ (protected)/
│  │  │  ├─ page.tsx          # dashboard
│  │  │  ├─ projects/ achievements/ experiences/ ...
│  ├─ api/                    # route handler (jika perlu)
│  ├─ layout.tsx
│  └─ sitemap.ts
├─ components/
│  ├─ ui/                     # button, input, card, dialog
│  ├─ public/                 # section publik
│  └─ admin/                  # table, form, uploader
├─ lib/
│  ├─ supabase/ (client.ts, server.ts, middleware.ts)
│  ├─ validations/            # skema Zod per modul
│  ├─ actions/                # Server Actions per modul
│  └─ utils.ts
├─ types/                     # tipe database
└─ middleware.ts              # proteksi /admin
docs/
├─ PROJECT.md                 # dokumen ini
├─ CHANGELOG.md
└─ decisions/                 # ADR (catatan keputusan teknis)
supabase/
└─ migrations/                # file SQL berurutan
```

---

## 9. Rencana Pengerjaan (Fase)

Status: `[ ]` belum | `[~]` sedang | `[x]` selesai

### Fase 0: Persiapan
- [ ] Buat repo GitHub dan branch `main` + `dev`
- [ ] Inisialisasi Next.js + TypeScript + Tailwind + ESLint
- [ ] Buat project Supabase, simpan kredensial di `.env.local`
- [ ] Buat `.env.example`, pastikan `.env.local` ada di `.gitignore`
- [ ] Hubungkan repo ke Vercel (preview deployment aktif)

**Selesai jika:** halaman kosong ter-deploy ke Vercel.

### Fase 1: Fondasi Database dan Auth
- [ ] Tulis migrasi tabel (7.1) dan fungsi `is_admin()`
- [ ] Aktifkan RLS dan buat policy setiap tabel
- [ ] Buat bucket `media` beserta policy
- [ ] Setup klien Supabase (browser, server, middleware)
- [ ] Halaman `/admin/login` dan proteksi route `/admin/*`
- [ ] Tambahkan akun admin ke tabel `admins`
- [ ] Generate tipe TypeScript dari skema

**Selesai jika:** login berhasil, akses `/admin` tanpa login diarahkan ke login,
dan pengunjung tidak bisa menulis data lewat API.

### Fase 2: Admin Panel (inti CRUD)
- [ ] Layout admin (sidebar, header, logout)
- [ ] Komponen reusable: DataTable, FormField, ImageUploader, ConfirmDialog, StatusBadge
- [ ] CRUD Profile (singleton)
- [ ] CRUD Projects
- [ ] CRUD Achievements
- [ ] CRUD Experiences
- [ ] CRUD Skills
- [ ] CRUD Education + Certifications
- [ ] CRUD IP Records
- [ ] CRUD Mentored Projects + Testimonials
- [ ] Reorder (sort_order) dan toggle draft/published/featured
- [ ] Dashboard ringkasan

**Selesai jika:** semua modul bisa ditambah, diubah, dihapus, dan diurutkan dari admin.

### Fase 3: Halaman Publik
- [ ] Layout publik (navbar, footer, responsif)
- [ ] Home: hero, featured project, featured prestasi
- [ ] About
- [ ] Projects (list + filter) dan detail `[slug]`
- [ ] Achievements
- [ ] Experience (timeline)
- [ ] Skills
- [ ] Education + Sertifikasi
- [ ] Intellectual Property
- [ ] Mentored + Testimoni
- [ ] Contact: form + simpan ke `contact_messages` + halaman `/admin/messages`
- [ ] Loading, empty state, dan halaman 404

**Selesai jika:** semua data dari admin tampil benar di sisi publik.

### Fase 4: Penyempurnaan
- [ ] SEO: metadata, OG image, `sitemap.ts`, `robots.txt`
- [ ] Optimasi gambar dan caching (`revalidate` / `revalidatePath` setelah CRUD)
- [ ] Aksesibilitas (kontras, alt text, navigasi keyboard)
- [ ] Anti-spam form kontak
- [ ] Dark mode (opsional)
- [ ] Pencarian dan filter lanjutan (opsional)

### Fase 5: Pengujian dan Rilis
- [ ] Uji manual alur utama (checklist di bagian 11)
- [ ] Audit Lighthouse (target di bagian 6)
- [ ] Review keamanan: RLS, env, bucket policy
- [ ] Isi konten asli semua modul
- [ ] Domain kustom (opsional) dan deploy produksi
- [ ] Tag rilis `v1.0.0`

### Fase 6: Pemeliharaan (berkelanjutan)
- [ ] Perbarui konten tiap semester
- [ ] Update dependency berkala
- [ ] Pastikan project Supabase tetap aktif

---

## 10. Standar Pengembangan

### 10.1 Git
- Branch: `main` (produksi), `dev` (integrasi), `feat/<nama>`, `fix/<nama>`
- Commit mengikuti Conventional Commits:
  `feat: tambah CRUD projects`, `fix: perbaiki upload gambar`,
  `docs: update PROJECT.md`, `chore: update dependency`
- Satu fitur = satu branch = satu Pull Request (walau kerja sendiri,
  biar riwayat rapi)

### 10.2 Kode
- TypeScript strict, hindari `any`
- Satu skema Zod per modul, dipakai di form dan Server Action
- Komponen kecil dan reusable; logika data dipisah dari tampilan
- Penamaan: file `kebab-case`, komponen `PascalCase`, variabel `camelCase`

### 10.3 Keamanan
- Jangan commit `.env.local`
- `SUPABASE_SERVICE_ROLE_KEY` hanya di server, jangan diawali `NEXT_PUBLIC_`
- Validasi ulang semua input di server
- Batasi tipe dan ukuran file upload

### 10.4 Definition of Done (per task)
- [ ] Fungsi berjalan sesuai kebutuhan
- [ ] Validasi dan penanganan error ada
- [ ] Responsif di mobile dan desktop
- [ ] Tidak ada error lint dan TypeScript
- [ ] Sudah dites manual
- [ ] Dokumentasi/Changelog diperbarui

---

## 11. Checklist Pengujian Manual

| # | Skenario | Hasil yang Diharapkan |
|---|----------|-----------------------|
| 1 | Akses `/admin` tanpa login | Diarahkan ke `/admin/login` |
| 2 | Login dengan kredensial salah | Muncul pesan error |
| 3 | Tambah project (draft) | Tidak tampil di publik |
| 4 | Ubah status ke published | Tampil di `/projects` |
| 5 | Upload gambar > batas ukuran | Ditolak dengan pesan jelas |
| 6 | Hapus data | Ada konfirmasi, data hilang dari publik |
| 7 | Kirim form kontak | Pesan muncul di `/admin/messages` |
| 8 | Insert langsung ke tabel dengan anon key | Ditolak oleh RLS |
| 9 | Buka di layar 360px | Layout tidak pecah |

---

## 12. Risiko dan Mitigasi

| Risiko | Dampak | Mitigasi |
|--------|--------|----------|
| Supabase free tier di-pause | Situs error | Health-check berkala, ingat cara resume |
| Kuota storage habis | Upload gagal | Kompres gambar sebelum upload, batasi ukuran |
| Scope membengkak | Tidak selesai | Patuhi prioritas P0 dulu, fitur P2 setelah rilis |
| Kebocoran key | Data rawan | Pisahkan env, rotasi key bila perlu |
| Data pribadi mahasiswa bimbingan | Masalah privasi | Wajib ada kolom izin tampil (`consent`) |

---

## 13. Dependensi Utama

```
next, react, typescript, tailwindcss
@supabase/supabase-js, @supabase/ssr
zod, react-hook-form, @hookform/resolvers
lucide-react, clsx, tailwind-merge
react-markdown (render deskripsi Markdown)
```

---

## 14. Environment Variables

```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=      # server only
NEXT_PUBLIC_SITE_URL=
```

---

## 15. Changelog

Format mengikuti Keep a Changelog; versi mengikuti Semantic Versioning.

### [Unreleased]
- Dokumen proyek dibuat

---

## 16. Catatan Keputusan (ADR Singkat)

| # | Keputusan | Alasan | Tanggal |
|---|-----------|--------|---------|
| 001 | Next.js App Router + Supabase | Gratis, satu ekosistem, mudah deploy | 2026-10-02 |
| 002 | Satu admin, tabel `admins` + RLS | Sederhana, aman | 2026-10-02 |