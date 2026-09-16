## Match Scanned Tag ID

```mermaid
%%{init: {'flowchart': {'curve': 'linear', 'nodeSpacing': 60, 'rankSpacing': 80, 'diagramPadding': 24}}}%%
flowchart TD
    N_START["Input: Array tag_id<br/>Lokasi = user login"]

    N_START --> N_MASTER["Cari Linen dari Tag ID"]
    N_MASTER --> N_FOUND{"Linen Ditemukan?"}
    N_FOUND -->|Tidak| N_UNREG["5 kategori<br/>Unregistered"]
    N_FOUND -->|Ya| N_CAND["Kandidat Linen terdaftar<br/>Ambil scan terakhir tiap tag<br/>berdasarkan last_activity_code"]

    N_START --> N_TRX["Cari 1 TRX terbaru<br/>di lokasi user<br/>urut created_at"]
    N_TRX --> N_HAS{"Ada transaksi?"}
    N_HAS -->|Tidak| N_NEW["accumulated_transaction_id kosong"]
    N_HAS -->|Ya| N_CHECK{"Sudah memiliki ISS atau PS?"}
    N_CHECK -->|Ya| N_NEW
    N_CHECK -->|Tidak| N_ACC["accumulated_transaction_id = ID transaksi<br/>DPS tidak menghentikan akumulasi"]
    N_ACC --> N_EXIST["Ambil tag pada OSS transaksi tersebut"]

    N_CAND --> N_OUT{"Scan terakhir = Outgoing Soil<br/>atau Driver Pickup Soil<br/>atau Driver Pickup Soil Scan?"}
    N_OUT -->|Ya| N_ALREADY["already_outgoing"]
    N_OUT -->|Tidak| N_INACC{"Tag sudah ada<br/>di OSS transaksi akumulasi?"}
    N_EXIST --> N_INACC
    N_INACC -->|Ya| N_ALREADY
    N_INACC -->|Tidak| N_LOC{"Lokasi Linen = Lokasi User<br/>atau di BAC(is_on_provider TRUE)?"}
    N_LOC -->|Ya| N_OTHERLOC["in_other_location"]
    N_LOC -->|Tidak| N_READY["registered<br/>Dikelompokkan per jenis linen"]

    N_READY --> N_RES["Respons:<br/>accumulated_transaction_id<br/>registered<br/>already_outgoing<br/>unregistered<br/>in_other_location"]
    N_ALREADY --> N_RES
    N_UNREG --> N_RES
    N_OTHERLOC --> N_RES
    N_ACC --> N_RES
    N_NEW --> N_RES

    class N_LOC,N_OTHERLOC danger
```

## Submit

```mermaid
%%{init: {'flowchart': {'curve': 'linear', 'nodeSpacing': 60, 'rankSpacing': 80, 'diagramPadding': 24}}}%%
flowchart TD
    N_IN["Payload:<br/>activity_code, activity_name, scan_device<br/>weight, seluruh response Proses Match<br/>accumulated_transaction_id"]
    N_IN -->N_REGIST["Match yang Registered"]
    N_REGIST -->N_MODE{"ID akumulasi terisi?"}

    N_IN -->N_ELSE["Match selain Registered"]
    N_ELSE -->N_PROC["Dilakukan Pencatatan di DB"]

    N_MODE -->|Tidak| N_NEW["Buat transaksi TRX baru<br/>lokasi = user login"]
    N_NEW --> N_ACT["Buat aktivitas OSS<br/>status aktivitas = SOIL<br/>berat dan biaya dari payload"]

    N_MODE -->|Ya| N_OLD["Gunakan transaksi akumulasi"]
    N_OLD --> N_ADD["Ambil aktivitas OSS existing<br/>Tambah berat dan hitung ulang biaya<br/>Tambah jumlah per jenis"]

    N_ACT --> N_DETAIL["Simpan rekap jenis linen<br/>dan rincian tag registered<br/>category = MATCHED, status = SOIL"]
    N_ADD --> N_DETAIL
    N_DETAIL --> N_SCAN["Catat scan Outgoing Soil<br/>lokasi = user login"]
    N_SCAN --> N_MASTER["Update master linen registered:<br/>status = SOIL<br/>is_on_provider = false<br/>location_id = lokasi user<br/>last_scaning_date dan updated_at<br/>last_activity_code, last_transaction_id<br/>last_transaction_location_id"]
    N_MASTER --> N_BILL["Sinkronkan billing harian<br/>jumlah tag + berat"]
    N_BILL --> N_DONE["Commit dan respons sukses"]

    N_IN -.-> N_TRUST["Submit tidak mengulang match:<br/>already_outgoing, In Other Location dan kelayakan<br/>akumulasi tidak diperiksa ulang"]

    class N_ELSE,N_PROC danger
```
