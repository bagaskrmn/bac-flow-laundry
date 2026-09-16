## Match Scanned Tag ID

```mermaid
flowchart TD
    N_IN["Input:<br/>transaction_id, Array tag_id<br/>scan_device dan activity_name opsional"]
    N_IN --> N_MASTER["Cari master + registrasi<br/>jenis, status, kondisi, kepemilikan"]
    N_MASTER --> N_UNREG["Tidak ditemukan<br/>unregistered"]

    N_IN --> N_MODE{"Jenis transaksi?"}
    N_MODE -->|TRX| N_OSS["Target = jumlah OSS per jenis"]
    N_OSS --> N_PART{"is_partial = true?"}
    N_PART -->|Ya| N_SUB["Kurangi detail request terkait<br/>Query tidak membatasi status Packed"]
    N_PART -->|Tidak| N_TARGET["Target jumlah"]
    N_SUB --> N_TARGET
    N_MODE -->|Request| N_REQ["Target = detail request<br/>ID tetap wajib ada di transactions"]
    N_REQ --> N_TARGET

    N_MODE -->|TRX| N_OWNER["Bandingkan belongs_to_location<br/>dengan lokasi transaksi"]
    N_MODE -->|Request| N_NOOWNER["Pemeriksaan pemilik dinonaktifkan<br/>missplaced = kosong"]
    N_MASTER --> N_OWNER
    N_OWNER --> N_WRONG{"Pemilik tidak null dan berbeda?"}
    N_WRONG -->|Ya| N_MIS["missplaced"]
    N_WRONG -->|Tidak| N_COND["Periksa kondisi"]
    N_NOOWNER --> N_COND
    N_COND --> N_GOOD["Layak:<br/>CLEAN + GOOD/null"]
    N_COND --> N_BAD["unproccessable_tag:<br/>bukan CLEAN atau WEAK"]

    N_GOOD --> N_COMP["Bandingkan jenis + jumlah"]
    N_TARGET --> N_COMP
    N_COMP --> N_MATCH["matched"]
    N_COMP --> N_ADD["additional:<br/>kelebihan atau jenis lain"]
    N_TARGET --> N_MISS["missing:<br/>kekurangan dari jumlah<br/>semua tag terdaftar per jenis"]
    N_MASTER --> N_MISS

    N_COMP -.-> N_NOTE["Tidak membandingkan tag dengan PS.<br/>Tidak ada already_packed<br/>atau in_other_transaction."]
```

## Submit

```mermaid
flowchart TD
    N_IN["Payload normal DPC:<br/>activity_code = DPC<br/>activity_name = Driver Pickup Clean<br/>transaction_id, scan_device, weight<br/>kategori hasil match"]

    N_IN --> N_ACCEPT["Proses empat kategori:<br/>matched + additional<br/>+ unproccessable_tag + missplaced"]
    N_ACCEPT --> N_ACT["Buat aktivitas<br/>kode mengikuti payload<br/>status aktivitas = CLEAN"]
    N_ACT --> N_QTY["Rekap jumlah seluruh empat kategori"]
    N_ACT --> N_TAG["Simpan rincian tag<br/>sesuai kategori"]
    N_TAG --> N_SCAN["Catat scan seluruh empat kategori<br/>Jika nama tepat Driver Pickup Clean:<br/>provider = provider user, lokasi = null"]

    N_QTY --> N_TRX["last_activity_code mengikuti payload"]
    N_SCAN --> N_TRX
    N_TRX --> N_MASTER["Update master untuk empat kategori:<br/>is_on_provider = true<br/>last_scaning_date dan updated_at"]
    N_MASTER --> N_DONE["Commit dan respons sukses"]

    N_ACCEPT -.-> N_RULE["Unprocessable dan misplaced<br/>tidak memblokir submit"]
    N_MASTER -.-> N_KEEP["Status, lokasi master,<br/>dan kepemilikan tidak diubah"]
```