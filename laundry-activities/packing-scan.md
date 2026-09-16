## Match Scanned Tag ID

Input: `type = OUT`, `id` transaksi atau request, `scan_device`, dan array `tag_id`.
**Target** dari tahap 1 serta **already_packed** dan **linen layak** dari tahap 2
menjadi masukan tahap 3.

### 1. Tentukan target

```mermaid
%%{init: {'flowchart': {'curve': 'stepAfter', 'nodeSpacing': 35, 'rankSpacing': 45}}}%%
flowchart TD
    N_SOURCE{"Sumber target?"}
    N_SOURCE -->|TRX| N_PART_PACKED{"TRX punya PART-PICKUP terkait<br/>yang berstatus Packed?"}
    N_PART_PACKED -->|Tidak| N_OSS_FULL["Jumlah OSS per jenis"]
    N_OSS_FULL --> N_TARGET["Target"]
    N_PART_PACKED -->|Ya| N_OSS["Jumlah OSS per jenis dikurangi<br/>jumlah PART-PICKUP terkait<br/>yang berstatus Packed per jenis<br/>Minimum 0"]
    N_SOURCE -->|Request| N_REQ["Jumlah detail request:<br/>HSREQ / NEWREQ-ADD<br/>NEWREQ-MS / PART-PICKUP"]
    N_OSS --> N_TARGET
    N_REQ --> N_TARGET
```

### 2. Periksa kandidat tag

```mermaid
%%{init: {'flowchart': {'curve': 'stepAfter', 'nodeSpacing': 35, 'rankSpacing': 45}}}%%
flowchart TD
    N_IN["Tag ID yang dipindai + Transaction ID terpilih"]
    N_IN --> N_MASTER["Cari registrasi + jenis linen<br/>status, kondisi, kepemilikan"]
    N_MASTER --> N_FOUND{"Registrasi ditemukan?"}
    N_FOUND -->|Tidak| N_UNREG["Tidak ditemukan<br/>unregistered"]
    N_FOUND -->|Ya| N_LAST["Baca scan terakhir tiap tag"]
    N_LAST --> N_OTHER{"Scan terakhir Packing Scan<br/>untuk transaksi lain?"}
    N_OTHER -->|Ya| N_O["in_other_transaction<br/>Keluarkan dari kandidat"]
    N_OTHER -->|Tidak| N_CAND["Kandidat berikutnya"]
    N_IN --> N_EXIST["Cari PS existing pada ID yang sama<br/>Ambil tag yang sudah dipacking"]
    N_EXIST --> N_WAS{"Kandidat sudah dipacking<br/>pada transaksi terpilih?"}
    N_CAND --> N_WAS
    N_WAS -->|Ya| N_PACKED["already_packed<br/>Keluarkan dari kandidat"]
    N_WAS -->|Tidak| N_OWNER{"Pemilik lokasi kosong atau<br/>sama dengan lokasi transaksi?"}
    N_OWNER -->|Tidak| N_MISPLACE["missplaced"]
    N_OWNER -->|Ya| N_CLEAN{"CLEAN dan kondisi GOOD?"}
    N_CLEAN -->|Ya| N_ELIGIBLE["Linen layak"]
    N_CLEAN -->|Tidak| N_BAD["Jika bukan CLEAN atau WEAK:<br/>unproccessable_tag"]
```

### 3. Bandingkan target dan hasil scan

Nama masukan di bawah merujuk ke keluaran tahap 1 dan 2.

```mermaid
%%{init: {'flowchart': {'curve': 'stepAfter', 'nodeSpacing': 35, 'rankSpacing': 45}}}%%
flowchart TD
    N_TARGET["Target<br/>dari tahap 1"] --> N_REDUCE["Missing awal<br/>Bandingkan Target dan Already Packed"]
    N_PACKED["already_packed<br/>dari tahap 2"] --> N_REDUCE
    N_ELIGIBLE["Linen layak<br/>dari tahap 2"] --> N_COMP["Bandingkan jenis dan kuota"]
    N_REDUCE --> N_COMP
    N_COMP --> N_MATCH["matched<br/>Jenis sesuai, dalam kuota"]
    N_COMP --> N_ADD["additional<br/>Melebihi kuota atau jenis tidak diminta"]

    N_COMP --> N_MISS["missing akhir<br/>max(0, missing awal - jumlah matched)"]
```

## Submit

```mermaid
flowchart TD
    N_IN["Payload hasil match<br/>type = OUT, id, weight<br/>Array matched, Array additional, Array missing<br/>Array unproccessable_tag, Array missplaced<br/>partial_packing_transaction_id"]

    N_IN --> N_BLOCK{"Unprocessable atau<br/>misplaced tidak kosong?"}
    N_BLOCK -->|Ya| N_REJECT["Ditolak<br/>Pesan: lakukan Scan Clean"]
    N_BLOCK -->|Tidak| N_MODE{"ID partial packing terisi?"}

    N_MODE -->|Ya| N_PART["Target = ID partial packing<br/>Ambil PS terbaru<br/>Tambah berat dan jumlah"]
    N_MODE -->|Tidak| N_SOURCE{"Sumber berupa request?"}
    N_SOURCE -->|Ya| N_REQ["Buat transaksi dari request<br/>lokasi = lokasi request<br/>Tandai request Packed<br/>Hubungkan parent jika partial pickup"]
    N_SOURCE -->|Tidak| N_NORMAL["Gunakan transaksi existing"]
    N_REQ --> N_CREATE["Buat aktivitas PS baru"]
    N_NORMAL --> N_CREATE

    N_PART --> N_SAVE["Simpan matched + additional<br/>Rekap jenis + rincian tag<br/>Scan Packing Scan"]
    N_CREATE --> N_SAVE
    N_SAVE --> N_TRX["last_activity_code = PS"]
    N_TRX --> N_MASTER["Update master:<br/>is_on_provider = true<br/>wash_cycle +1<br/>updated_at"]
    N_MASTER --> N_DONE["Commit dan respons sukses"]
```

## Commit Packing

```mermaid
flowchart TD
    N_IN["User/Admin memilih transaction yang sudah dipacking untuk dicommit"]
    N_IN --> N_LIST["Transaksi yang dapat dicommit hanya yang sudah ter-submit packing < target<br/>Jika submit packing > target auto ter-commit<br/>User dapat input Tipe Linen dan QTY untuk melengkapi hutang Packing yang sudah dicommit"]
```
