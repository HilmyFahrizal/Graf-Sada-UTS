# UTS Graf Sains Data: Rekomendasi Berbasis Graf (RetailRocket + Neo4j)

| Nama | NIM | Kelas |
|---|---|---|
| Muh. Hilmy Fahrizal | 23031554125 | 2023D |

Sistem rekomendasi berbasis graf dari data interaksi toko online RetailRocket. Data disimpan sebagai graf di Neo4j, dianalisis per periode waktu, dipakai untuk merekomendasikan item dengan KNN, lalu dievaluasi dengan Hit@5 dan NDCG@5.

## Filter periode yang dipakai

Seluruh data `events.csv` diimpor ke Neo4j. Periode waktu hanya berfungsi sebagai **filter interaksi** pada properti `timestamp` (unix epoch, milidetik, UTC). Batas akhir bersifat **eksklusif**.

| Periode | Rentang tanggal | Peran | Jumlah baris |
|---|---|---|---:|
| **P1** | 1 sampai 15 Agustus 2015 | training (membangun graf, VSM, similarity, KNN) | 265.509 |
| **P2** | 16 sampai 20 Agustus 2015 | testing (ground truth evaluasi) | 95.404 |

| Variabel | Unix timestamp (ms) | Waktu (UTC) |
|---|---:|---|
| `P1_start` | `1438387200000` | 2015-08-01 00:00:00 |
| `P1_end` | `1439683200000` | 2015-08-16 00:00:00 |
| `P2_start` | `1439683200000` | 2015-08-16 00:00:00 |
| `P2_end` | `1440115200000` | 2015-08-21 00:00:00 |

Syarat filter: `batas_awal <= timestamp < batas_akhir`. Syarat yang sama dipakai di pandas dan di Cypher (`WHERE r.timestamp >= $ts AND r.timestamp < $te`).

## Isi repositori

| File | Isi |
|---|---|
| `requirements.txt` | Dependensi proyek (`neo4j>=5.7`, `pandas`, `tqdm`, `scikit-learn`, `scipy`, `networkx`, `matplotlib`, dan paket bawaan soal) |
| `UTS_Pendahuluan.ipynb` | Bagian 1.1 dan 1.2: persiapan lingkungan, tutorial driver Neo4j (`add_friend`, `print_friends`), dan jawaban tiga pertanyaan *checkpoint* |
| `Import_RetailRocket.ipynb` | Bagian 1.3: impor seluruh `events.csv` ke Neo4j per batch |
| `UTS_RetailRocket.ipynb` | Bagian 2 (soal UTS): 2.1 Wrangling dan EDA, 2.2 Analytics |
| `data/events_train.csv` | Hasil filter P1 (1 sampai 15 Agustus 2015) |
| `data/events_test.csv` | Hasil filter P2 (16 sampai 20 Agustus 2015) |
| `checkpoint1.png` sampai `checkpoint3.png` | Bukti jawaban *checkpoint* (dipakai `UTS_Pendahuluan.ipynb`) |

`events.csv` (data asli, sekitar 94 MB) tidak disertakan. Unduh dari [Kaggle: RetailRocket ecommerce-dataset](https://www.kaggle.com/datasets/retailrocket/ecommerce-dataset) dan letakkan di folder `data/`.

## Model graf

```
(:Visitor)-[:VIEWED]->(:Item)
(:Visitor)-[:ADDED_TO_CART]->(:Item)
(:Visitor)-[:BOUGHT]->(:Item)
```

Satu baris data menjadi satu relasi berarah. Setiap relasi menyimpan `timestamp`, dan `BOUGHT` menyimpan juga `transactionId`.

## Ringkasan hasil

| Bagian | Hasil utama |
|---|---|
| Komposisi subgraph P1 | 153.836 `Visitor`, 76.078 `Item`, 265.509 relasi. Graf bipartit, 96,6% relasi `VIEWED` |
| VSM | User × Item 153.836 × 76.078 dan Item × User 76.078 × 153.836, sparsity 99,9982% |
| Proyeksi terpilih | **Item–Item** (lebih efisien dan lebih efektif daripada User–User) |
| Similarity | cosine biner. Item acuan 45337, lima item paling mirip ditulis sebagai relasi `SIMILAR_TO` |
| Rekomendasi | KNN item-based, k = 5. User acak 123595, tiga item iklan: 113754, 132335, 168546 |
| Evaluasi user 123595 | Hit@5 = 0 dan NDCG@5 = 0 |
| Evaluasi 200 user acak | rata-rata Hit@5 = 0,045 dan NDCG@5 = 0,022 |

## Cara menjalankan

1. Buat dan aktifkan *virtual environment* (`python -m venv myenv`), lalu jalankan Neo4j secara lokal di `neo4j://localhost:7687`.
2. Pasang dependensi: `pip install -r requirements.txt`. File ini memakai `neo4j>=5.7` (bukan `neo4j` 4.x pada soal) karena `execute_query` baru tersedia di driver 5.x.
3. Letakkan `events.csv` di folder `data/`.
4. Set password Neo4j (diminta saat sel dijalankan):
   - Git Bash / Linux / macOS: `export NEO4J_PASSWORD="password_kamu"`
   - PowerShell: `$env:NEO4J_PASSWORD="password_kamu"`
5. Jalankan notebook **berurutan**, masing-masing dari atas ke bawah:
   1. `UTS_Pendahuluan.ipynb`
   2. `Import_RetailRocket.ipynb` (mengisi graf, cukup sekali)
   3. `UTS_RetailRocket.ipynb`

Paket yang dibutuhkan dipasang pada sel `%pip install` di tiap notebook: `neo4j` (5.7 atau lebih baru), `pandas`, `tqdm`, `scikit-learn`, `scipy`, `networkx`, dan `matplotlib`.
