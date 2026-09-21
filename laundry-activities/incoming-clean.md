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
    N_IN["Input:<br/>transaction_id, Array tag_id<br/>scan_device"]
    N_IN --> N_MASTER{"Linen dan tipe ditemukan?"}
    N_MASTER -->|Tidak| N_UNREG["Tidak ditemukan<br/>unregistered"]

    N_IN --> N_OSS["Target = Tag ID pada PS<br/>di Transaction terpilih"]
    N_MASTER -->|Ya| N_COMP["Bandingkan Tag ID"]
    N_OSS --> N_COMP

    N_COMP -->N_MS["Missing: Ada di Target<br/>tapi tidak ada di Input"]
    N_COMP -->N_MATCH["Matched: Ada di Target<br/>dan di Input"]
    N_COMP -->N_PS{"Apakah aktivitas terakhir PS/DPC?"}
    N_PS -->|Ya| N_LASTPS["For Other Transactions"]
    N_PS -->|Tidak| N_LASTNOTPS["Not Packed"]

    class N_PS,N_LASTPS,N_LASTNOTPS danger

```

## Submit

```mermaid
%%{init: {'flowchart': {'curve': 'linear', 'nodeSpacing': 60, 'rankSpacing': 80, 'diagramPadding': 24}}}%%
flowchart TD
    N_IN["Payload transaction_id, scan_device<br/>Semua data hasil match"]

    N_IN -->N_NOTES["Proses Selain Matched"]
    N_NOTES --> N_CATAT["Catat ke DB"]
    N_CATAT --> N_MISSING["MISSING dibuatkan NEWREQ-MS"]

    N_IN --> N_ACCEPT["Proses matched"]
    N_ACCEPT --> N_TRX["Update Transaksi"]
    N_TRX --> N_ACT["Buat aktivitas<br/>status aktivitas = CLEAN"]
    N_ACT --> N_QTY["Rekap jumlah seluruh linen"]
    N_QTY --> N_TAG["Simpan rincian tag<br/>dan insert history linen"]
    N_TAG --> N_SCAN["update linen<br/>lokasi, status, updated_at,<br/>last_activity_code, last_transaction_id<br/>last_transaction_location_id"]

    N_SCAN --> N_DONE["Commit dan respons sukses"]
```
    
```