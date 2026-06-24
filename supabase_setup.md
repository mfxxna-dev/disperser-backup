# Panduan Setup Supabase - Disperser Studio 🎙️

Dokumen ini berisi panduan langkah-demi-langkah untuk menyiapkan Database dan Storage Bucket pada project **Supabase baru** agar sinkron dengan aplikasi Disperser Studio.

---

## 1. Setup Database Tables

Buka **SQL Editor** di dashboard Supabase Anda, buat query baru, lalu salin dan jalankan (Run) SQL script di bawah ini:

```sql
-- ==========================================
-- 1. PEMBUATAN TABEL USERS
-- ==========================================
CREATE TABLE IF NOT EXISTS public.users (
    id TEXT PRIMARY KEY, -- Menggunakan Discord User ID (contoh: '1501273981720727652')
    username TEXT NOT NULL,
    avatar TEXT,
    current_role TEXT DEFAULT 'Free',
    last_login TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc'::text, now()),
    roblox_user_id TEXT,
    roblox_api_key TEXT,
    uploads_today INTEGER DEFAULT 0,
    last_upload_date TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc'::text, now()),
    subscription_expires_at TIMESTAMP WITH TIME ZONE
);

-- ==========================================
-- 2. PEMBUATAN TABEL TRANSACTIONS (PEMBAYARAN DUITKU)
-- ==========================================
CREATE TABLE IF NOT EXISTS public.transactions (
    merchant_order_id TEXT PRIMARY KEY,
    user_id TEXT REFERENCES public.users(id) ON DELETE CASCADE,
    role_target TEXT NOT NULL,
    status TEXT DEFAULT 'pending',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc'::text, now())
);

-- ==========================================
-- 3. PEMBUATAN TABEL AUDIO LIBRARY (RIWAYAT UPLOAD)
-- ==========================================
CREATE TABLE IF NOT EXISTS public.audio_library (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    description TEXT,
    status TEXT DEFAULT 'pending',
    file_path TEXT,
    user_id TEXT REFERENCES public.users(id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc'::text, now()),
    asset_id TEXT,
    operation_path TEXT,
    error_message TEXT
);

-- ==========================================
-- 4. NONAKTIFKAN ROW LEVEL SECURITY (RLS) UNTUK KEMUDAHAN
-- ==========================================
-- Karena aplikasi frontend mengakses database secara langsung menggunakan anonymous key
-- tanpa otentikasi Supabase Auth (menggunakan Discord OAuth kustom),
-- kita perlu menonaktifkan RLS agar frontend dapat membaca/menulis data.

ALTER TABLE public.users DISABLE ROW LEVEL SECURITY;
ALTER TABLE public.transactions DISABLE ROW LEVEL SECURITY;
ALTER TABLE public.audio_library DISABLE ROW LEVEL SECURITY;
```

---

## 2. Setup Storage Bucket (`audios`)

Untuk menyimpan file audio sementara yang di-upload dari lokal sebelum diteruskan ke Roblox, Anda perlu membuat bucket di Supabase Storage.

### Langkah-langkah:
1. Buka menu **Storage** di dashboard Supabase Anda.
2. Klik tombol **New Bucket**.
3. Beri nama bucket: **`audios`** *(harus persis huruf kecil semua)*.
4. Aktifkan opsi **Public bucket** (agar backend dapat mengunduh audio menggunakan public URL, atau untuk mempermudah signed URL).
5. Klik **Create bucket**.

### Kebijakan Akses (Policies) Bucket:
Supabase Storage memerlukan kebijakan akses (policies) agar aplikasi dapat meng-upload dan menghapus file audio.
Di dalam detail bucket `audios`, buka tab **Policies** (atau buka **Storage > Policies**):
1. **Untuk kemudahan testing**, Anda dapat menambahkan kebijakan baru dengan template **"Get full access (Read, Write, Update, Delete) to all users"** (pilih untuk role `anon` dan `authenticated`).
2. Jika ingin dikustomisasi lewat SQL Editor, Anda bisa menjalankan script SQL ini untuk memberikan akses penuh kepada pengguna anonim/umum:

```sql
-- Kebijakan INSERT untuk Bucket audios
CREATE POLICY "Allow public uploads" ON storage.objects 
FOR INSERT TO anon, authenticated 
WITH CHECK (bucket_id = 'audios');

-- Kebijakan SELECT untuk Bucket audios
CREATE POLICY "Allow public reads" ON storage.objects 
FOR SELECT TO anon, authenticated 
USING (bucket_id = 'audios');

-- Kebijakan UPDATE untuk Bucket audios
CREATE POLICY "Allow public updates" ON storage.objects 
FOR UPDATE TO anon, authenticated 
USING (bucket_id = 'audios');

-- Kebijakan DELETE untuk Bucket audios
CREATE POLICY "Allow public deletes" ON storage.objects 
FOR DELETE TO anon, authenticated 
USING (bucket_id = 'audios');
```

---

## 3. Update Environment Variables (`.env`)

Setelah database dan bucket siap, jangan lupa untuk memperbarui file `.env` di root project Anda dengan credential Supabase yang baru:

```env
SUPABASE_DB_PASSWORD=Password_Database_Supabase_Baru
VITE_SUPABASE_URL=https://id_project_baru.supabase.co
VITE_SUPABASE_ANON_KEY=anon_public_key_baru
```

Pastikan Anda melakukan **restart backend** dan **restart frontend** agar environment variables yang baru ter-load dengan sempurna.

```bash
# Restart Backend
cd backend && npm run dev

# Restart Frontend
cd frontend && npm run dev
```
