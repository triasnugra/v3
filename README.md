Hasil v2 sudah masuk, dan jujur saja: **v2 belum lebih baik dari v1**. Skornya turun dari 47% ke 44%, dan trade-nya sendiri juga memburuk di beberapa tempat. Ini bukan kesimpulan dari sampel kecil. Ada pola yang bisa dilacak ke kode.

## Yang berhasil

- **Short palsu di Monday High hilang.** SOL-0815, LTC-0726, dan SOL-0625 tidak lagi menghasilkan short. Itu sesuai dugaan.
- **Engine tidak lagi menyebut tengah range saat Eliz bilang bukan.** Di BTC-0827, v2 hanya memasang order tunda di ekstrem, dan itu dinilai "sama". Selain itu, v2 mengenali level weekly di beberapa koin (ALICE, TURBO, BTC) yang sebelumnya tidak ada.
- **Penilaian adil bekerja.** Kecocokan arah dan pola sekarang masuk akal.

## Yang memburuk, dan kenapa

**1. Breakdown/breakout trendline di 1D menghasilkan setup yang sering salah arah dan tidak masuk akal.**
Di v2, `tlb` aktif dan mengalahkan Wolf di banyak tempat. Contohnya:
- FET-0917: short 1D dengan entry 0,169 saat harga 0,159 dan bias harian naik. Eliz bullish. Setup ini kena SL.
- SOL-0712: short 4H dengan entry 77,85 saat Eliz dan bias harian sama-sama bullish. Kena SL.
- ETH-0620: long 1D di entry 1564 saat harga 1703. Ini bukan retest, melainkan entry 8% di bawah harga. Order tunda seperti ini seharusnya tidak lolos.
- FLOKI-0618: masih short, padahal kedua engine tetap berlawanan dengan Eliz.

Ada dua masalah teknis di sini. Pertama, fungsi `tb` menghitung "retest" sebagai harga berada dalam 2×ATR dari garis, tanpa memastikan harga benar-benar kembali ke garis. Kedua, pada 1D, garis trendline dari dua pivot dengan `c-a>=5` sering sangat curam atau terlalu jauh, sehingga ln(T,n) melenceng dari harga sekarang dan "retest" berarti hal yang berbeda.

**2. Mismatch sisi entry dan tren.** Mengizinkan `tlb` tanpa melihat arah entry relatif terhadap harga sekarang membuat order tunda yang jauh di bawah harga. Itu sebabnya ETH-0620 menghasilkan entry 1564 untuk harga 1703 dan tetap diterima.

**3. Sampel tidak cukup.** Pembanding acak juga memburuk: rata-rata 0,18R untuk v2 dibanding -0,05R untuk v1. Median sama-sama -1R, dan 53 trade terisi dengan 14 menang. Angka rata-rata v2 terdorong satu trade besar (FLOKI +8,7R, ALICE +7,6R). Tanpa trade terbaik, v2 sebenarnya cukup mirip v1. Jadi yang bisa kita simpulkan hanya bahwa v2 belum terbukti lebih baik.

## Yang kusarankan, berurutan

Aku tidak menyarankan perbaikan besar lagi sebelum satu hal diuji dengan benar.

1. **Matikan `tlb` dulu (flag, bukan dihapus).** Di v2 hanya setup itu yang paling jelas merusak hasil. Lalu ukur ulang. Kalau skornya kembali ke 47–50% atau naik, berarti masalahnya memang di `tlb`.
2. **Kalau `tlb` diaktifkan kembali,** perbaiki dua hal: harga sekarang harus sudah bolak-balik ke garis (bukan hanya dekat), dan entry harus berada di sisi yang sama dengan harga, jadi order tunda yang lebih dari 2% dari harga ditolak.
3. **Tambah jumlah sampel.** Dataset 21 post tidak cukup untuk membedakan 44% dari 47%. Pembanding acak 100 sampel pun masih terlalu berisik untuk selisih sekecil itu. Angka yang bisa dipercaya butuh ratusan sampel atau backtest penuh di seluruh riwayat.

Mau aku langsung lakukan langkah 1 dan 2 (matikan `tlb` dengan flag, lalu perbaiki pemeriksaan retest dan jarak entry)? Setelah itu kamu jalankan ulang tab Uji, dan kita lihat apakah v2 akhirnya lebih baik dari v1.

Sekarang tab Uji: tiga skor berdampingan, plus baris khusus untuk sampel yang hasilnya berbeda antar versi.Terakhir, jalankan seluruh pipeline tiga varian dengan server Binance palsu, lalu cek bahwa fungsi ringkasan dan tampilan tidak crash.Pipeline tiga varian jalan tanpa crash. Dari hasil terakhirmu aku juga menemukan hal yang harus kamu perbaiki di sisi deploy, dan versi baru ini kutambahi penanda versi supaya itu kelihatan di layar.Langkah 1 dan 2 sudah jadi, tapi belum diuji dengan data Binance asli, jadi belum ada bukti bahwa v2 sekarang lebih baik dari v1.

## Ada masalah di hasil terakhirmu

`eliz-dataset.json` yang terpasang waktu itu masih versi lama. Buktinya BTC-0729 masih tertulis "context, dilewati" dan yang dinilai 17 entri, padahal dataset baru menilai 18. Jadi perbandingan 44% vs 47% yang kamu tempel dihitung dengan dataset yang berbeda dari yang kukira. Kali ini deploy **keempat** file. Layar akan menampilkan baris "Terpasang: harness v3, dataset v2 dengan 21 entri". Kalau tulisannya "LAMA" atau dataset bukan v2, ada file yang belum terganti.

## Yang berubah

- **`tlb` mati secara default**, dikendalikan flag `ctx.tlb`.
- **Aturan `tlb` diperketat.** Entry limit di garis harus dalam 2,5% dari harga. Harga juga harus pernah kembali ke dalam 0,5 ATR dari garis setelah candle break, atau sedang berada di sana. Dua kasus buruk dari hasilmu (FET short dengan entry 6,7% dari harga, ETH long dengan entry 8% di bawah harga) akan ditolak aturan ini.
- **Tiga varian pada data yang sama:** v1 (lama), v2 (`tlb` mati), dan v2t (`tlb` aktif dengan aturan ketat). Dengan begitu efek `tlb` terpisah dari perubahan lainnya.
- **Pembanding acak jadi 200 sampel**, plus satu baris khusus untuk sampel yang setup-nya berbeda antar versi. Di sampel yang identik selisihnya nol dan hanya menambah derau.

## Yang sudah kuuji

- Mode legacy tetap identik dengan v1 asli (60 dari 60).
- `tlb` mati di default, dan aktif hanya lewat flag.
- Geometri yang sama menghasilkan setup kalau wick kembali ke garis dan tidak kalau tidak. Harga yang jauh dari garis ditolak.
- Setup non-`tlb` tidak berubah sama sekali dibanding v2 sebelumnya (60 dari 60).
- Di data acak, setup `tlb` turun dari 8 ke 2, dan yang tersisa semuanya subset dari yang lama.
- Seluruh pipeline tiga varian jalan tanpa crash dengan server Binance palsu. Itu hanya menguji alur kode, bukan kualitas hasil.

## Cara membaca hasil nanti

- **v2 vs v1:** efek perubahan sapuan Monday dan "tengah range" saja. Harapanku v2 setidaknya tidak lebih buruk dari v1 di sini, karena `tlb` sudah mati. Itu dugaan, bukan jaminan.
- **v2t vs v2:** efek `tlb` yang sudah diperbaiki. Kalau v2t tidak lebih baik dari v2 pada sampel yang berbeda, `tlb` kita tinggalkan mati.
- **Lihat baris "sampel yang berbeda"**, median, dan "tanpa trade terbaik", bukan rata-rata. Di uji acak sebelumnya, rata-rata v2 terdorong dua trade besar.
- **Selisih kecil jangan dibaca sebagai bukti.** Dengan 18 entri dataset dan beberapa puluh sampel yang berbeda, selisih satu atau dua poin masih bisa kebetulan.

## Yang belum kuubah

Wolf, Double Deviation, dan lebar SL tidak kusentuh. Short Wolf 1D masih jadi setup terbaik di FLOKI, BTC-0712, dan SOL-0625, dan semuanya dinilai berbeda dari Eliz. Kalau v2 dan v2t sudah stabil, itu kandidat berikutnya, dan kuusulkan juga batas jarak entry untuk semua jenis setup, bukan hanya `tlb`.

Jalankan uji dataset lalu uji pembanding acak (200 sampel butuh beberapa menit), dan tempel hasilnya.
