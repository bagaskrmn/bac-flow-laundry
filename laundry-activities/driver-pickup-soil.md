## Match Scanned Tag ID

```mermaid
flowchart TD
    N_IN["Input:<br/>transaction_id<br/>match_with<br/>Array tag_id"]

    N_IN --> N_MASTER["Cari linen terdaftar<br/>EPC + master + registrasi"]
    N_MASTER --> N_REGISTERED["Tag scan yang ditemukan"]
    N_MASTER --> N_UNREG["Tag tidak ditemukan<br/>unregistered"]

    N_IN --> N_BASE["Ambil tag aktivitas transaksi<br/>activity_code = match_with<br/>OSS"]
    N_BASE --> N_COMP["Bandingkan tag ID"]
    N_REGISTERED --> N_COMP

    N_COMP --> N_MATCH["registered<br/>Ada di baseline dan discan"]
    N_COMP --> N_MISS["missing<br/>Ada di baseline, tidak discan"]
    N_COMP --> N_CHECK_ADD{"location_id = transaction.location_id<br/>dan is_on_provider = false?"}

    N_CHECK_ADD -->|Ya| N_NOT_OUT["not_outgoing"]
    N_CHECK_ADD -->|Tidak| N_ADD["additional<br/>Terdaftar, di luar baseline"]

    N_MATCH --> N_GROUP["Kelompokkan per jenis:<br/>linen_type_id, name, count, Array tag_id"]
    N_MISS --> N_GROUP
    N_ADD --> N_GROUP
    N_NOT_OUT --> N_GROUP
    N_GROUP --> N_RES["Respons kategori"]
    N_UNREG --> N_RES

    N_IN -.-> N_RULE["Transaksi dipilih client.<br/>Tidak memeriksa status,<br/>kondisi tag, atau kepemilikan."]
```

## Submit

```mermaid
flowchart TD
    N_IN["Payload:<br/>activity_code = DPS<br/>transaction_id, activity_name<br/>scan_device, weight<br/>Array registered, Array missing, Array additional"]
    N_IN --> N_ACT["Update transaksi menjadi DPS<br/>Buat aktivitas DPS / SOIL<br/>Berat dari payload"]

    N_ACT --> N_QTY["Rekap jumlah per jenis<br/>registered + additional"]
    N_ACT --> N_LOG["Riwayat linen:<br/>registered → MATCHED<br/>missing → MISSING<br/>additional → ADDITIONAL"]

    N_QTY --> N_REAL["Tag scan aktual:<br/>registered + additional"]
    N_LOG --> N_REAL
    N_REAL --> N_SCAN["Catat activity_scan_tag<br/>nama = activity_name dari client<br/>lokasi = lokasi transaksi"]
    N_SCAN --> N_MASTER["Update master tag aktual:<br/>status = SOIL<br/>is_on_provider = false<br/>location_id = lokasi transaksi<br/>last_scaning_date dan updated_at"]
    N_MASTER --> N_DONE["Commit dan respons sukses"]

    N_LOG -.-> N_MISS["Missing hanya riwayat:<br/>tidak update master<br/>tidak dibuatkan request pengganti"]
```