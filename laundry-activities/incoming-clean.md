## Get List Transaction

```mermaid
%%{init: {'flowchart': {'curve': 'linear', 'nodeSpacing': 60, 'rankSpacing': 80, 'diagramPadding': 24}}}%%
flowchart TD
    N_IN["Hanya berisi transactions yang memiliki PS,<br/>tidak memiliki IC, dan Commited"]

```

## Match Scanned Tag ID

```mermaid
%%{init: {'flowchart': {'curve': 'linear', 'nodeSpacing': 60, 'rankSpacing': 80, 'diagramPadding': 24}}}%%
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
%%{init: {'flowchart': {'curve': 'linear', 'nodeSpacing': 60, 'rankSpacing': 80, 'diagramPadding': 24}}}%%
flowchart TD
    N_IN["Payload normal IC:<br/>activity_code = IC<br/>activity_name = Incoming Clean<br/>transaction_id, scan_device, weight<br/>kategori hasil match"]

    N_IN --> N_ACTUAL["Tag aktual yang diproses:<br/>matched + additional"]
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
    
```