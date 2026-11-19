# Cancel Packing

## Cakupan

## Cancel Packing

Input: `tag_id` (scan maupun ketik manual) + reason

```mermaid
%%{init: {'flowchart': {'curve': 'linear', 'nodeSpacing': 60, 'rankSpacing': 80, 'diagramPadding': 24}}}%%
flowchart TD
    N_IN["Input Tag"]
    N_IN --> N_VALIDATE["Cek Aktivitas dan Transaksi Terakhir masin2 tag"]
    N_VALIDATE --> N_VALDPC["Terakhir DPC"]
    N_VALDPC --> N_VALDPCACT["Dilakukan oleh Risk Management<br/>Opsi bisa putihkan linen saja atau<br/>hapus dari aktivitas PS dan DPC"]
    

    N_VALIDATE --> N_VALPSCOMM["Terakhir PS SUDAH Commit"]
    N_VALPSCOMM --> N_VALPSCOMMACT["Dilakukan oleh Risk Management<br/>Opsi bisa putihkan linen saja atau<br/>hapus dari aktivitas PS"]


    N_VALIDATE --> N_VALPS["Terakhir PS BELUM Commit"]
    N_VALPS --> N_VALPSACT["Putihkan Linen<br/>Hapus Linen dari Transaksi"]
   
```

## Discussions

- historis linen dicatat
- setiap tag -1 Wash Cycle
- Mencatat Transaksi pembuatan Cancel Packing
- Jika melakukan PS SUDAH Commmit dan/atau DPC maka commit harus di-cancel
