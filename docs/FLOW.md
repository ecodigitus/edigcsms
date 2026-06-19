# Edigcsms — Bagan Alir & Keterhubungan Sistem

> Dokumen kerja untuk validasi ke konsultan CSMS/e-CHSEMS.
> Acuan: **PTK 005 / kriteria e-CHSEMS SKK Migas**. Angka ambang & detail portal perlu dikonfirmasi ke edisi terbaru.

---

## 1. Bagan alir end-to-end (siapa mengerjakan apa)

Aliran dari pendaftaran vendor sampai status "lulus CSMS". Warna lane menandai pemilik proses.
Zona **Edigcsms** = bagian persiapan yang selama ini dibebankan ke konsultan berbayar.

```mermaid
flowchart TD
  subgraph V["VENDOR / MITRA KERJA"]
    A0["0 · Registrasi di CIVD<br/>data & legalitas perusahaan"]
  end

  subgraph E["EDIGCSMS — produk kita (persiapan & readiness)"]
    A1["1 · Klasifikasi risiko pekerjaan<br/>rendah / sedang / tinggi"]
    A2["2 · Isi kuesioner 8 elemen<br/>+ kumpulkan bukti pendukung"]
    A3["3 · Self-scoring & gap analysis<br/>target lulus ≥ 60% / ≥ 54,3%"]
    PKG["Paket PK siap-submit"]
  end

  subgraph K["PORTAL & TIM PENILAI KKKS"]
    A4["4 · Submit ke portal resmi<br/>PHE VMS / PEP i-P2P"]
    A5["5 · Verifikasi dokumen + lapangan<br/>lalu skoring oleh penilai"]
  end

  subgraph S["SKK MIGAS (sistem pemerintah)"]
    A6["6 · Input Nilai Kualifikasi K3LL<br/>ke e-CHSEMS (oleh KKKS)"]
    CIVD[("CIVD<br/>database vendor terpusat")]
    A7["7 · Lulus → boleh ikut tender<br/>berlaku lintas KKKS · lanjut PB/PA"]
  end

  A0 --> A1 --> A2 --> A3 --> PKG
  PKG -. "di-upload manual oleh vendor" .-> A4
  A4 --> A5 --> A6 --> A7
  A0 -. tertaut .- CIVD
  A6 -. terintegrasi .- CIVD

  classDef edig fill:#FDEFDD,stroke:#C2630B,stroke-width:2px,color:#16201D;
  classDef gov  fill:#E7F0F7,stroke:#2B5F8A,color:#16201D;
  classDef kkks fill:#E2F0EE,stroke:#0E6E66,color:#16201D;
  classDef pass fill:#E4F2E9,stroke:#2E7D52,color:#16201D;
  class A1,A2,A3,PKG edig;
  class A0,A6,CIVD gov;
  class A4,A5 kkks;
  class A7 pass;
```

**Titik kunci:** Edigcsms berhenti tepat di **"Paket PK siap-submit"**. Penyerahan ke portal KKKS dilakukan **manual oleh vendor** — tidak ada koneksi otomatis ke sistem pemerintah.

---

## 2. Keterhubungan sistem — di mana Edigcsms menempel

Garis **tegas** = integrasi nyata yang bisa kita bangun.
Garis **putus-putus** = serah-terima manual / tidak ada API publik (sistem pemerintah).

```mermaid
flowchart LR
  vendor["Vendor (pengguna)"]

  subgraph EDIG["EDIGCSMS Platform"]
    wiz["Wizard kuesioner<br/>8 elemen"]
    repo["Repository dokumen<br/>(reusable lintas tender)"]
    score["Mesin scoring<br/>& gap analysis"]
    exp["Export paket PK<br/>(PDF / arsip)"]
  end

  %% --- integrasi yang bisa dibangun (solid) ---
  vendor --> wiz
  wiz --> repo --> score --> exp
  auth["Auth / SSO"] --> EDIG
  store["Cloud storage<br/>terenkripsi"] --- repo
  pay["Payment<br/>(langganan SaaS)"] --- EDIG
  sign["e-Sign dokumen"] -. opsional .- exp
  expert["Expert review<br/>(layanan hybrid)"] -. opsional .- score

  %% --- serah-terima manual ke sistem pemerintah (dashed, no API) ---
  exp -. "upload manual oleh vendor" .-> portal["Portal KKKS<br/>PHE VMS / PEP i-P2P"]
  portal -. hasil dinilai .-> echs["e-CHSEMS<br/>(SKK Migas)"]
  echs -. terintegrasi .- civd["CIVD<br/>(SKK Migas)"]

  classDef edig fill:#FDEFDD,stroke:#C2630B,stroke-width:2px,color:#16201D;
  classDef ext  fill:#F3F5F1,stroke:#4A554F,color:#16201D;
  classDef gov  fill:#E7F0F7,stroke:#2B5F8A,stroke-dasharray:4 3,color:#16201D;
  class wiz,repo,score,exp edig;
  class auth,store,pay,sign,expert,vendor ext;
  class portal,echs,civd gov;
```

### Ringkasan keterhubungan

| Sistem | Hubungan dengan Edigcsms | Jenis |
|--------|--------------------------|-------|
| **CIVD** (SKK Migas) | Tidak terhubung langsung — vendor daftar sendiri | ❌ Tidak ada API |
| **Portal KKKS** (PHE VMS / PEP i-P2P) | Tujuan akhir paket — vendor upload manual | ⚠️ Serah-terima manual |
| **e-CHSEMS** (SKK Migas) | Hanya menerima nilai dari KKKS | ❌ Tidak ada API |
| **Auth / SSO** | Login & manajemen akun vendor | ✅ Integrasi internal |
| **Cloud storage terenkripsi** | Simpan bukti dokumen (data sensitif) | ✅ Integrasi internal |
| **Payment** | Langganan SaaS | ✅ Integrasi pihak ketiga |
| **e-Sign** (opsional) | Tanda tangan dokumen kebijakan | ✅ Integrasi pihak ketiga |
| **Expert review** (opsional) | Layanan hybrid berbayar | ✅ Modul internal |

---

## 3. Urutan interaksi (sequence)

```mermaid
sequenceDiagram
  actor V as Vendor
  participant ED as Edigcsms
  participant PK as Portal KKKS
  participant TP as Tim Penilai KKKS
  participant EC as e-CHSEMS (SKK Migas)

  V->>ED: Isi kuesioner 8 elemen + unggah bukti
  ED-->>V: Skor prediksi + daftar gap (≥60% / ≥54,3%)
  V->>ED: Perbaiki gap sampai siap
  ED-->>V: Paket PK siap-submit (PDF/arsip)
  V->>PK: Upload paket (manual)
  PK->>TP: Teruskan untuk penilaian
  TP->>TP: Verifikasi dokumen + lapangan + skoring
  TP->>EC: Input Nilai Kualifikasi K3LL
  EC-->>V: Status "lulus CSMS" (berlaku lintas KKKS)
```

---

## Catatan kepatuhan & keamanan

- **Batas produk:** Edigcsms = layer *persiapan/readiness*. Bukan penerbit sertifikat, bukan pengganti portal resmi/penilai KKKS.
- **Tidak overclaim:** output = *kesiapan*, bukan jaminan lulus.
- **Keamanan data (OWASP):** dokumen vendor sangat sensitif (legalitas, data personel) → enkripsi at-rest & in-transit, kontrol akses berbasis peran, audit log. Wajib jadi prioritas sejak desain.
