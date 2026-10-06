## Match Scanned Tag ID

```mermaid
%%{init: {'flowchart': {'curve': 'linear', 'nodeSpacing': 60, 'rankSpacing': 80, 'diagramPadding': 24}}}%%
flowchart TD
    N_IN["Input:<br/>type = IN<br/>id = ID lokasi/RS<br/>scan_device, Array tag_id"]

    N_IN --> N_LOOKUP["Cari linen dari tag_id"]

    N_LOOKUP --> N_UNREG["Tag tidak ditemukan<br/>unregistered"]
    N_LOOKUP --> N_CHECK_ISS{"last_activity_code == ISS?"}

    N_CHECK_ISS -->|Ya| N_ALREADY["already_incoming_soil"]
    N_CHECK_ISS -->|Tidak| N_CHECK_ACT{"last_activity_code != OSS / DPS?"}

    N_CHECK_ACT -->|Ya| N_CHECK_PROV{"Lokasi di BAC"}
    N_CHECK_PROV -->|Ya| N_OTHERLOC["from_other_location"]
    N_CHECK_PROV -->|Tidak| N_CHECK_LOC2{"linen location_id == req location_id?"}
    N_CHECK_LOC2 -->|Ya| N_ADD_NEWREQ["Additional<br/>Lanjut ke NEWREQ-ADD"]
    N_CHECK_LOC2 -->|Tidak| N_OTHERLOC

    N_CHECK_ACT -->|Tidak| N_HISTORY["Hubungan transaksi:<br/>tag → linen_activities<br/>→ laundry_activities → transactions"]

    N_HISTORY --> N_FILTER{"location_transaction == location_id request?"}
    N_FILTER -->|Ya| N_IDS["Ambil transaction_id unik"]
    N_FILTER -->|Tidak| N_OTHERLOC
    N_IDS --> N_BASE["Baseline:<br/>tag kategori MATCHED<br/>pada aktivitas OSS<br/>dari transaksi yang ditemukan"]

    N_BASE --> N_EXISTS{"Baseline ditemukan?"}
    N_EXISTS -->|Tidak| N_EMPTY["HTTP 200<br/>No transaction found<br/>data = null"]
    N_EXISTS -->|Ya| N_COMP["Bandingkan baseline OSS<br/>dengan semua tag terdaftar yang discan"]

    
    N_COMP -->N_CEKINC{"Apakah Tag sudah ter-incoming soil<br/>di transaksi tersebut?"}
    N_CEKINC -->|Ya|N_ALREADY
    N_CEKINC -->|Tidak| N_MATCHED["Linen Matched Final"]

    N_COMP -.-> N_ADDNOTE["Catatan: Hasil ini tidak lagi ada Additional<br/>karena masuk ke from_other_location"]
    N_UNREG --> N_RES["Response proses match ISS"]
    N_ALREADY --> N_RES
    N_OTHERLOC --> N_RES
    N_ADD_NEWREQ --> N_RES
    N_MATCHED --> N_RES

    class N_CHECK_ISS,N_CHECK_ACT,N_CEKINC,N_ALREADY,N_CHECK_PROV,N_CHECK_LOC2,N_OTHERLOC,N_ALREADY danger
```

## Submit

```mermaid
%%{init: {'flowchart': {'curve': 'linear', 'nodeSpacing': 60, 'rankSpacing': 80, 'diagramPadding': 24}}}%%
flowchart TD
    N_IN["Payload:<br/>location_id,scan_device<br/>Seluruh data proses Match"]
    N_IN -->N_MATCHED["Matched"]
    N_MATCHED -->N_UPDTRX["Update Transactions"]
    N_MATCHED -->N_LAUNDLIN["Insert/Update Laundry Linens"]
    N_MATCHED -->N_LA["Insert Linen Activities"]
    N_MATCHED -->N_AST["Insert Activity Scan Tag"]
    N_MATCHED -->N_UPDLIN["Update Linen Lists"]


    N_IN -->N_ALREADYINC["Already Incoming Soil"]
    N_ALREADYINC -->N_AST
    N_ALREADYINC -->N_UPDLIN

    N_IN -->N_ADD["Additional"]
    N_ADD -->N_NEWREQADD["Create New Req ADD"]
    N_ADD -->N_AST
    N_ADD -->N_UPDLIN

    N_IN -->N_FROMOTHER["From Other Location"]
    N_FROMOTHER -->N_AST
    N_FROMOTHER -->N_UPDLIN

```

## 1 Scan 2 Transaksi dengan lokasi yang sama

```mermaid
%%{init: {'flowchart': {'curve': 'linear', 'nodeSpacing': 60, 'rankSpacing': 80, 'diagramPadding': 24}}}%%
flowchart LR
    N_T1["OSS TRX-1<br/>A, B"]
    N_T2["OSS TRX-2<br/>C, D"]
    N_S["Scan ISS<br/>A, C, X"]

    N_T1 --> N_COMP["Match"]
    N_T2 --> N_COMP
    N_S --> N_COMP

    N_COMP --> N_M["Matched: A, C"]
    N_COMP --> N_MS["Missing: B, D"]
    N_COMP --> N_AD["Additional: X"]

    N_M --> N_I1["ISS TRX-1<br/>A diterima, B missing"]
    N_MS --> N_I1
    N_M --> N_I2["ISS TRX-2<br/>C diterima, D missing"]
    N_MS --> N_I2
    N_AD --> N_R["NEWREQ-ADD<br/>untuk X"]
```
