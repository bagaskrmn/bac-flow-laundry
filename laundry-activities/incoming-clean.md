## Match Scanned Tag ID

```mermaid
flowchart TD
    N_IN["Input IC:<br/>transaction_id + Array tag_id"]
    N_IN --> N_SHARED["Fungsi match yang sama dengan DPC"]
    N_SHARED --> N_SOURCE{"Sumber target?"}
    N_SOURCE -->|TRX| N_OSS["Jumlah OSS<br/>dikurangi detail request jika partial"]
    N_SOURCE -->|Request| N_REQ["Jumlah detail request"]

    N_SHARED --> N_MASTER["Registrasi tag<br/>status + kondisi + pemilik"]
    N_MASTER --> N_OWNER["Cek pemilik pada TRX saja<br/>Request: misplaced kosong"]
    N_OWNER --> N_GOOD["Layak = CLEAN + GOOD/null"]
    N_OWNER --> N_OTHER["misplaced / unproccessable_tag"]
    N_MASTER --> N_UNREG["unregistered"]

    N_OSS --> N_COMP["Bandingkan jenis dan jumlah"]
    N_REQ --> N_COMP
    N_GOOD --> N_COMP
    N_COMP --> N_RES["matched / additional / missing"]
    N_OTHER --> N_OUT["Respons IC"]
    N_UNREG --> N_OUT
    N_RES --> N_OUT

    N_COMP -.-> N_NOTE["Tidak memakai hasil DPC sebagai baseline.<br/>Tidak wajib tag yang sama dengan OSS/PS/DPC.<br/>Tidak memastikan sudah PS atau DPC."]
```

## Submit

```mermaid
flowchart TD
    N_IN["Payload normal IC:<br/>activity_code = IC<br/>activity_name = Incoming Clean<br/>transaction_id, scan_device, weight<br/>kategori hasil match"]

    N_IN --> N_ACTUAL["Tag aktual yang diproses:<br/>matched + additional<br/>+ unproccessable_tag + missplaced"]
    N_ACTUAL --> N_ACT["Buat aktivitas CLEAN<br/>Rekap jumlah + rincian tag"]
    N_ACT --> N_SCAN["Catat scan Incoming Clean<br/>lokasi = lokasi transaksi<br/>provider = null"]
    N_SCAN --> N_TRX["last_activity_code = IC"]
    N_TRX --> N_MASTER["Update master tag aktual:<br/>is_on_provider = false<br/>location_id = lokasi transaksi<br/>last_scaning_date dan updated_at"]

    N_IN --> N_HASMISS{"Missing ada?"}
    N_HASMISS -->|Ya| N_ID["Bentuk NEWREQ-MS<br/>mengikuti ID transaksi asal"]
    N_ID --> N_REQ["Buat request pengganti<br/>status = Approved<br/>sumber = Missing Incoming Clean<br/>lokasi = lokasi transaksi"]
    N_REQ --> N_DETAIL["Detail request:<br/>linen_type_id + quantity missing"]
    N_HASMISS -->|Tidak| N_DONE["Commit dan respons sukses"]
    N_DETAIL --> N_DONE
    N_MASTER --> N_DONE

    N_MASTER -.-> N_NOTE["Status master dan kepemilikan tetap.<br/>Tag misplaced bisa dipindahkan lokasi master<br/>tanpa mengubah pemiliknya."]
    N_ACTUAL -.-> N_ACCEPT["Unprocessable dan misplaced<br/>tidak memblokir submit"]
    
```