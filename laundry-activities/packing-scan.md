## Match Scanned Tag ID

```mermaid
flowchart TD
    N_IN["Input:<br/>type = OUT<br/>id = transaksi atau request<br/>scan_device, Array tag_id"]

    N_IN --> N_MASTER["Cari registrasi + jenis linen<br/>status, kondisi, kepemilikan"]
    N_MASTER --> N_UNREG["Tidak ditemukan<br/>unregistered"]
    N_MASTER --> N_LAST["Baca scan terakhir tiap tag"]
    N_LAST --> N_OTHER{"Scan terakhir Packing Scan<br/>untuk transaksi lain?"}
    N_OTHER -->|Ya| N_O["in_other_transaction<br/>Keluarkan dari kandidat"]
    N_OTHER -->|Tidak| N_CAND["Kandidat berikutnya"]

    N_IN --> N_SOURCE{"Sumber target?"}
    N_SOURCE -->|TRX| N_OSS["Jumlah OSS per jenis<br/>dikurangi request terkait<br/>yang statusnya Packed"]
    N_SOURCE -->|Request| N_REQ["Jumlah detail request:<br/>HSREQ / NEWREQ-ADD<br/>NEWREQ-MS / PART-PICKUP"]
    N_OSS --> N_TARGET["Target awal"]
    N_REQ --> N_TARGET

    N_IN --> N_EXIST["Cari PS existing pada ID yang sama<br/>Ambil tag yang sudah dipacking"]
    N_EXIST --> N_REDUCE["Kurangi target dengan<br/>jumlah packing existing"]
    N_TARGET --> N_REDUCE
    N_EXIST --> N_WAS{"Kandidat sudah dipacking?"}
    N_CAND --> N_WAS
    N_WAS -->|Ya| N_PACKED["already_packed<br/>Keluarkan dari kandidat"]
    N_WAS -->|Tidak| N_OWNER{"Pemilik lokasi tidak null<br/>dan berbeda dari target?"}
    N_OWNER -->|Ya| N_MISPLACE["missplaced"]
    N_OWNER -->|Tidak| N_CLEAN{"CLEAN dan kondisi GOOD/null?"}
    N_CLEAN -->|Ya| N_ELIGIBLE["Linen layak"]
    N_CLEAN -->|Tidak| N_BAD["Jika bukan CLEAN atau WEAK:<br/>unproccessable_tag"]

    N_ELIGIBLE --> N_COMP["Bandingkan jenis dan kuota"]
    N_REDUCE --> N_COMP
    N_COMP --> N_MATCH["matched<br/>Jenis sesuai, dalam kuota"]
    N_COMP --> N_ADD["additional<br/>Melebihi kuota atau jenis tidak diminta"]

    N_REDUCE --> N_MISS["missing<br/>Target dikurangi jumlah kandidat<br/>terdaftar per jenis"]
    N_CAND -.-> N_MISS
    N_MISPLACE -.-> N_NOTE["Missing menghitung kandidat terdaftar,<br/>termasuk yang tidak layak.<br/>Missing 0 belum berarti semua layak."]
    N_BAD -.-> N_NOTE
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
    N_DATE --> N_DONE["Commit dan respons sukses"]
    N_SKIP --> N_DONE
```