## Match Scanned Tag ID

```mermaid
flowchart TD
    IN["Input:<br/>transaction_id<br/>match_with<br/>tag_id[]"]

    IN --> MASTER["Cari linen terdaftar<br/>EPC + master + registrasi"]
    MASTER --> REGISTERED["Tag scan yang ditemukan"]
    MASTER --> UNREG["Tag tidak ditemukan<br/>unregistered"]

    IN --> BASE["Ambil tag aktivitas transaksi<br/>activity_code = match_with<br/>OSS"]
    BASE --> NOTE["Baseline tidak dibatasi<br/>hanya category MATCHED"]
    BASE --> COMP["Bandingkan tag ID"]
    REGISTERED --> COMP

    COMP --> MATCH["registered<br/>Ada di baseline dan discan"]
    COMP --> MISS["missing<br/>Ada di baseline, tidak discan"]
    COMP --> ADD["additional<br/>Terdaftar, di luar baseline"]

    MATCH --> GROUP["Kelompokkan per jenis:<br/>linen_type_id, name, count, tag_id[]"]
    MISS --> GROUP
    ADD --> GROUP
    GROUP --> RES["Respons kategori"]
    UNREG --> RES

    IN -.-> RULE["Transaksi dipilih client.<br/>Tidak memeriksa status,<br/>kondisi tag, atau kepemilikan."]
```

## Submit

```mermaid
flowchart TD
    IN["Payload:<br/>activity_code = DPS<br/>transaction_id, activity_name<br/>scan_device, weight<br/>registered[], missing[], additional[]"]
    IN --> CHECK{"Kode DPS?"}
    CHECK -->|Tidak| REJECT["Ditolak"]
    CHECK -->|Ya| ACT["Update transaksi menjadi DPS<br/>Buat aktivitas DPS / SOIL<br/>Berat dari payload"]

    ACT --> QTY["Rekap jumlah per jenis<br/>registered + additional"]
    ACT --> LOG["Riwayat linen:<br/>registered → MATCHED<br/>missing → MISSING<br/>additional → ADDITIONAL"]

    QTY --> REAL["Tag scan aktual:<br/>registered + additional"]
    LOG --> REAL
    REAL --> SCAN["Catat activity_scan_tag<br/>nama = activity_name dari client<br/>lokasi = lokasi transaksi"]
    SCAN --> MASTER["Update master tag aktual:<br/>status = SOIL<br/>is_on_provider = false<br/>location_id = lokasi transaksi<br/>last_scaning_date dan updated_at"]
    MASTER --> DONE["Commit dan respons sukses"]

    LOG -.-> MISS["Missing hanya riwayat:<br/>tidak update master<br/>tidak dibuatkan request pengganti"]
```