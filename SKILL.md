---
name: kecoak
description: >
  Membasmi bug seperti membasmi kecoak: sampai ke sarangnya, bukan cuma yang
  kelihatan — supaya tidak muncul lagi. Disiplin diagnosis + fix permanen dua jalur:
  Bug Rusak (fungsional: reproduksi dulu, diagnosis berhipotesis, cari akar,
  petakan dampak, kunci dengan regression test, visualisasi graf) dan Security Hardening (threat
  model ringkas, fix di trust boundary, verifikasi adversarial, sweep pola
  sejenis). Dipakai saat bug yang "sudah di-fix" muncul lagi, saat ada temuan
  keamanan, atau sebelum menyatakan "sudah beres".
  Trigger chat: "bug muncul lagi", "sudah di-fix tapi", "kecoak", "fix permanen",
  "regression", "ada celah", "security hardening", "bisa di-bypass".
license: MIT
version: 1.0.0
---

# kecoak

> Membasmi bug seperti membasmi kecoak: semprot yang kelihatan saja = besok
> muncul lagi. Harus sampai ke sarangnya. Skill ini memaksa disiplin itu.

## Fase 0 — Klasifikasi: Bug Rusak vs Security (wajib pertama)

Tentukan dulu jenisnya. Cara fix dan cara verifikasinya beda — salah klasifikasi
= fix yang salah.

| Sinyal | Jalur |
|---|---|
| Crash, output salah, fitur tidak jalan, UI rusak, lambat | **A — Bug Rusak** |
| Auth/authz, injection (SQL/OS/command), XSS, CSRF, IDOR, path traversal, SSRF, secret bocor, data sensitif terekspos, endpoint bisa di-abuse | **B — Security Hardening** |
| Ragu-ragu | Anggap **B** sampai terbukti A. Mengira vuln sebagai bug biasa jauh lebih mahal daripada sebaliknya. |

Contoh jebakan klasifikasi: pesan error menampilkan stack trace (kelihatan bug
kosmetik → sebenarnya info disclosure); tombol admin disembunyikan tapi endpoint-nya
terbuka (kelihatan bug UI → sebenarnya broken access control).

## Jalur A — Bug Rusak: loop diagnosis + fix permanen

### A1. Reproduksi dulu — no repro, no fix

- Buat **satu perintah loop** yang bisa dijalankan agent: merah saat bug ada,
  hijau saat beres, assert gejala persis seperti laporan (bukan sekadar "tidak crash").
- Ketatkan loop: cepat (detik, bukan menit), deterministik (pin waktu/seed bila
  perlu). Loop 30 detik yang flaky hampir sama buruknya dengan tidak ada loop.
- Tidak bisa reproduksi? **Berhenti.** Catat apa yang sudah dicoba, minta artefak
  (log, HAR, rekaman langkah) atau akses ke environment tempat bug muncul. Jangan
  menebak fix tanpa repro — ini penyebab nomor satu "sudah di-fix" yang fiktif.
- Bug kadang-kadang: naikkan reproduction rate (loop 100x, stress, persempit
  timing) sampai cukup sering untuk di-debug.

### A2. Diagnosis: hipotesis, bukan tebakan

- Buat **3–5 hipotesis ber peringkat** sebelum menguji satu pun. Satu hipotesis
  = terpaku pada ide pertama yang masuk akal.
- Tiap hipotesis harus falsifiable: "Jika X penyebabnya, maka mengubah Y akan
  menghilangkan bug / mengubah Z akan memperparah." Tidak bisa dirumuskan =
  bukan hipotesis, buang.
- Tunjukkan daftar ke user sebelum menguji (mereka sering punya info yang
  me-rerank instan, mis. "kemarin baru deploy ubahan di situ"). Jangan blokir
  kalau user AFK — lanjut dengan peringkatmu.
- Uji **satu variabel per satu**. Instrumen: debugger/REPL dulu (satu breakpoint
  > sepuluh log), lalu log ter-target di batas yang membedakan hipotesis.
  Jangan "log semuanya lalu grep".
- Tag semua log debug dengan prefix unik (mis. `[DBG-xxxx]`) — cleanup nanti
  tinggal satu grep.
- **Dilarang melompat ke fix sebelum hipotesis terkonfirmasi.** Di sinilah bug
  "abadi" lahir.
- → Update bugraph: rantai penyebab terisi (hipotesis terkonfirmasi = garis penuh).

### A3. Kunci akar + sweep saudara kandung

- Akar = penyebab yang kalau diubah, gejalanya hilang **dan** tidak punya saudara
  kandung. Uji: "kalau fix ini saya revert, apakah bug persisnya kembali?"
  Kalau tidak, itu perban — bukan akar.
- **Sweep saudara kandung:** cari pola yang sama di tempat lain (fungsi serupa,
  copy-paste, cabang kode paralel). Satu akar sering punya banyak gejala — obati
  semua, bukan satu.

### A4. Petakan dampak (blast radius)

- Sebelum mengubah: siapa yang memanggil fungsi ini? State apa yang dibagi?
  (Call graph ala `vulture` membantu di sini.)
- Daftarkan jalur yang terpengaruh. Fix yang merusak tempat lain = bug baru yang
  "tiba-tiba muncul" minggu depan.
- → Update bugraph: blast radius terisi.

### A5. Fix minimal + kunci dengan regression test

- Perubahan terkecil yang menyentuh akar masalah. Jangan refactor sekalian —
  refactor = blast radius baru.
- Tulis regression test **dulu** (merah), lalu fix, lalu hijau. Test harus menangkap
  pola bug yang sebenarnya, bukan versi yang disederhanakan sampai kehilangan makna.
- Kalau tidak ada seam yang tepat untuk test, catat sebagai temuan: arsitektur
  menghalangi bug ini dikunci. Itu utang yang harus dibayar, bukan alasan untuk skip.

### A6. Verifikasi

- [ ] Skenario repro asli (bukan yang diminimalkan) sudah hijau
- [ ] Regression test hijau
- [ ] Test suite terkait tetap hijau (fix tidak merusak yang lain)
- [ ] Gejala saudara kandung (A3) sudah dicek satu per satu
- [ ] File bugraph HTML tersimpan di `~/workspace/bugraph/<nama-kasus>/` dan
      sinkron dengan temuan terakhir

### A7. Catat + bersih-bersih

- Commit message menyebut **akar masalah dan hipotesis yang terbukti benar**,
  bukan cuma gejalanya — debugger berikutnya belajar dari sini.
- Kalau bug ini pernah "di-fix" sebelumnya, catat apa yang salah dari fix lama.
- Hapus semua log `[DBG-xxxx]` (grep prefix) dan prototipe throwaway.

## Jalur B — Security Hardening: disiplin adversarial

Bug security bukan "fitur yang rusak" — ini **celah yang bisa dieksploitasi**.
Cara berpikirnya beda: berpikir seperti penyerang, bukan seperti pengguna.

### B1. Model ancaman ringkas (5 menit, jangan di-skip)

- Siapa penyerang? (user anonim, user login, insider, bot)
- Apa yang mereka kontrol? (semua input — anggap input = musuh)
- Apa dampak kalau berhasil? (baca data orang lain, ubah data, RCE, dsb.)
- Di mana trust boundary-nya? (pintu masuk: API, form, URL param, file upload, webhook)

### B2. Fix di trust boundary, bukan di dalam

- Validasi / sanitasi / authorize **di pintu masuk**, sekali, konsisten. Fix yang
  tersebar di dalam logika = ada jalur yang kelewat.
- Pola yang benar: allowlist > denylist; parameterized query > escaping manual;
  authorize di server > sembunyikan tombol di UI.

### B3. Jangan obati gejala (aturan keras)

- Menyembunyikan tombol admin ≠ memperbaiki IDOR.
- Try/catch di sekitar query ≠ memperbaiki injection.
- Pesan error generik tanpa memperbaiki validasi ≠ memperbaiki apa-apa.
- Uji: "kalau penyerang memanggil langsung endpoint/fungsi ini, apakah masih
  tembus?" Kalau ya, belum fix.

### B4. Sweep pola sejenis (wajib)

- Celah keamanan **jarang sendirian**. Pola yang sama (query tanpa parameter,
  endpoint tanpa authz check, upload tanpa validasi) hampir pasti ada di tempat
  lain. Cari dan perbaiki semua — satu per satu, terverifikasi.

### B5. Verifikasi adversarial

- Buktikan exploit-nya **gagal**, bukan cuma buktikan fitur normal jalan. Tulis PoC
  test: kirim payload jahat → harus ditolak/dinetralkan.
- [ ] PoC exploit gagal (sebelum fix: berhasil — buktikan dengan test merah dulu)
- [ ] Jalur normal tetap berfungsi (tidak over-block sampai fitur mati)
- [ ] Tidak ada jalur bypass (parameter lain, method lain, endpoint kembar)

### B6. Catat dengan aman

- Commit message: sebut kategori + lokasi fix, **jangan** tulis detail exploit
  yang bisa dipakai orang (mis. "fix IDOR di /api/orders" cukup; jangan tulis
  payload-nya).
- Proyek open source + celah kritis: koordinasikan disclosure dulu, jangan umbar.

## Visualisasi bug / bugraph (wajib, bukan hiasan)

Setelah A2–A4 (atau B1/B4), sajikan analisis sebagai **graf** lewat HTML widget
(`widget.create_html_widget`) — bukan teks panjang. Dua graf:

1. **Rantai penyebab** — simpul: gejala → penyebab antara → **akar** (tandai beda
   warna). Panah = "disebabkan oleh".
   Contoh: `Error 500 di /checkout` → `order.total = null` →
   `diskon tak dihitung saat kupon expired` → **akar: tanpa guard kupon expired**.
2. **Blast radius** — simpul tengah = fungsi yang diubah; panah keluar = pemanggil
   / fitur yang terpengaruh. Tiap simpul terdampak wajib dicentang "terverifikasi".

**Aturan graf:**
- Tiap simpul = satu fakta terverifikasi. Yang masih hipotesis: garis putus-putus.
- Untuk Jalur B, tambah lapis "jalur serangan": simpul input penyerang → trust
  boundary → dampak. Boundary yang belum di-fix ditandai merah.

**Kapan di-render (terikat fase — bukan opsional):**
- A2 selesai → rantai penyebab terisi (hipotesis terkonfirmasi = garis penuh).
- A4 selesai → blast radius terisi.
- A6 → checklist verifikasi: graf sinkron dengan temuan terakhir.
- B4/B5 → lapis jalur serangan terisi.

**Output ganda (otomatis, tanpa diminta user):**
- Widget di chat — untuk dibaca sekarang.
- File HTML: `~/workspace/bugraph/<nama-kasus>/bugraph-<YYYYMMDD-HHMM>.html` —
  arsip yang bisa dibuka di browser atau dibagikan ke tim. Dibuat otomatis
  setiap sesi kecoak; update berikutnya menimpa file yang sama (satu kasus =
  satu file, selalu versi terakhir).

**Hemat token (aturan keras):**
- Render graf **sekali per sesi** (template HTML + data simpul/edge terpisah).
- Update berikutnya: kirim **hanya delta** (simpul/edge baru atau berubah), bukan
  render ulang penuh.
- Temuan kecil dalam fase yang sama dikumpulkan — render ulang maksimal sekali
  per fase.
- Sesi panjang: cabang yang sudah terverifikasi bersih boleh disembunyikan
  (collapse), bukan dihapus.

## Anti-pattern (yang bikin bug "abadi")

- Fix tanpa repro ("kayaknya ini penyebabnya, coba saya benerin").
- Test yang hijau padahal tidak menangkap pola bug aslinya (false confidence).
- Satu fix untuk satu gejala padahal akarnya punya lima gejala.
- Refactor besar dibungkus sebagai "fix bug" (blast radius meledak).
- Security: denylist yang tidak lengkap, validasi hanya di client, secret di kode,
  "tombolnya sudah saya sembunyikan".
- "Di laptop saya aman" untuk bug yang cuma muncul di production / data asli.

## Do & Don't

| ✅ DO | ❌ DON'T |
|---|---|
| Reproduksi dulu sebelum menyentuh kode | Fix berdasarkan tebakan tanpa repro |
| Kejar sampai akar; buktikan dengan revert-test | Tambah null check tanpa tahu kenapa null |
| Sweep pola yang sama di seluruh codebase | Satu fix untuk satu gejala padahal akarnya punya lima gejala |
| Fix minimal — jangan refactor sekalian | Refactor besar dibungkus sebagai "fix bug" |
| Kunci dengan regression test (merah → hijau) | Percaya test yang tidak menangkap pola bug aslinya |
| Petakan blast radius sebelum mengubah | Ubah fungsi shared tanpa cek pemanggilnya |
| Security: berpikir seperti penyerang | Security: "tombolnya sudah saya sembunyikan" |
| Security: allowlist, parameterized query, authorize di server | Security: denylist tak lengkap, validasi cuma di client |
| Security: buktikan exploit gagal (PoC jahat) | Security: cuma buktikan fitur normal jalan |
| Commit: tulis akar masalah, tanpa detail exploit | Commit: "fix bug" tanpa penjelasan / umbar payload |

## Kapan skill ini dipakai

- Dipicu dari chat: "bug muncul lagi", "sudah di-fix tapi", "kecoak", "fix permanen",
  "regression", "ada celah", "security hardening", "bisa di-bypass".
- Otomatis oleh agent: setiap diminta memperbaiki bug → mulai dari Fase 0.
  Setiap temuan security → Jalur B penuh, bukan versi singkat.

## Relasi opsional dengan skill lain (bukan dependensi)

Skill ini mandiri — bisa dipakai penuh tanpa skill lain. Daftar di bawah hanya
pintasan bila skill tersebut kebetulan ada, bukan syarat.

- Butuh peta dampak antar fungsi? Pakai `vulture` (call graph) untuk A4 / sweep B4.
- Optimasi performa setelah bug beres? Baru pakai `app-optimizer`.

## Kompatibilitas

Markdown murni, SKILL.md format standar. Tanpa hooks, plugin, MCP, atau script.
Install: copy folder `tuntas/` ke direktori skill masing-masing harness.
