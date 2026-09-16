## Get List Transaction

```mermaid
flowchart TD
    N_IN["Hanya berisi transactions yang memiliki PS,<br/>tidak memiliki DPC, dan Commited"]

```

## Match Scanned Tag ID

```mermaid
flowchart TD
    N_IN["Input:<br/>transaction_id, Array tag_id<br/>scan_device dan activity_name opsional"]
    N_IN --> N_MASTER{"Linen dan tipe ditemukan?"}
    N_MASTER -->|Tidak| N_UNREG["Tidak ditemukan<br/>unregistered"]

    N_IN --> N_OSS["Target = Tag ID pada PS<br/>di Transaction terpilih"]
    N_MASTER -->|Ya| N_COMP["Bandingkan Tag ID"]
    N_OSS --> N_COMP

    N_COMP -->N_MS["Missing: Ada di Target<br/>tapi tidak ada di Input"]
    N_COMP -->N_MATCH["Matched: Ada di Target<br/>dan di Input"]
    N_COMP -->N_ADD["Additional: Tidak ada di Target<br/>tapi ada di Input"]

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
