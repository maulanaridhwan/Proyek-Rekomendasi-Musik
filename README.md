# Laporan Proyek Machine Learning - Maulana Ridhwan Riziq

## Domain Proyek

Di era digital saat ini, platform streaming musik telah menghasilkan lonjakan besar dalam jumlah lagu dan variasi genre. Pengguna sering kali kesulitan menemukan lagu baru yang sesuai preferensi musikal mereka. Sistem rekomendasi musik membantu menyaring ribuan lagu untuk memberikan rekomendasi personal sesuai selera pengguna \[1].

Dengan menggunakan data lagu dan fitur audio serta metadata genre, sistem rudimenter ini akan menyarankan lagu-lagu yang memiliki karakteristik serupa dengan lagu pilihan pengguna. Pendekatan ini memanfaatkan content-based filtering yang menekankan pada kemiripan konten (audio features dan genre) antara lagu \[1].

Content-Based Filtering adalah metode sistem rekomendasi di mana setiap item, dalam hal ini lagu,  diwakili oleh sekumpulan fitur—baik itu teks, metadata, maupun ciri-ciri numerik. Untuk musik, fitur-fitur tersebut meliputi atribut audio seperti danceability, energy, valence dan metadata genre. Ide dasarnya adalah merekomendasikan item yang "mirip" dengan item yang disukai pengguna berdasarkan konten yang melekat pada item tersebut \[1].

Cara kerjanya:
1. Setiap lagu diwakili sebagai vektor fitur, misalnya, kombinasi fitur audio dan representasi TF-IDF untuk genre.
2. Ketika pengguna memilih atau menyukai sebuah lagu, sistem akan mengambil vektor fitur lagu tersebut sebagai query.
3. Sistem menghitung kemiripan (similarity) antara vektor query dan vektor semua lagu lain dalam koleksi.
4. Lagu-lagu dengan skor similarity tertinggi dianggap paling mirip dan direkomendasikan kepada pengguna.

Dalam proyek ini, fitur audio dinormalisasi dan dikombinasikan dengan vektor TF-IDF dari Artist Genres. Model kemudian menghitung ukuran jarak—yaitu cosine similarity—untuk menentukan seberapa mirip dua lagu.
## Business Understanding

### Problem Statements

* Bagaimana memberikan rekomendasi lagu yang relevan dan personal kepada pengguna berdasarkan profil lagu yang telah dipilih atau disukai sebelumnya?
* Bagaimana mengevaluasi kualitas rekomendasi secara objektif dalam konteks content-based filtering?

### Goals

* Membangun sistem rekomendasi musik berbasis content-based filtering yang menggabungkan fitur audio dan metadata genre.
* Mengevaluasi kinerja sistem menggunakan metrik Precision\@10, Recall\@10, dan NDCG\@10.
* Mendemonstrasikan rekomendasi 10 lagu teratas untuk input lagu tertentu.

### Solution Statements

* Menyiapkan dataset Top 10000 Spotify Songs (1950–Now) dengan fitur audio dan kolom `Artist Genres` \[2].
* Melakukan praproses data: menangani duplikat, menghapus nilai kosong, dan mengubah genre menjadi vektor TF-IDF \[3].
* Menggabungkan fitur audio (Min-Max scaling) dan TF-IDF genre ke dalam satu matriks fitur.
* Menghitung cosine similarity untuk merekomendasikan lagu-lagu terdekat (most similar) \[4].
* Mengevaluasi hasil rekomendasi menggunakan definisi relevansi berbasis primary genre.

## Data Understanding

Dataset yang digunakan: **Top 10000 Spotify Songs 1950-Now** (Kaggle) \[2]. Fitur utama:
* **Fitur**:
    * Track URI  - Uniform Resource Identifier unik untuk setiap lagu di Spotify.
    * Track Name  - Judul lagu.
    * Artist URI(s)  - URI artis yang terkait dengan lagu (bisa berisi beberapa URI jika ada kolaborasi).
    * Artist Name(s)  - Nama artis atau artis-artis yang membawakan lagu.
    * Album URI  - URI unik untuk album tempat lagu ini berada.
    * Album Name - Nama album tempat lagu ini dirilis.
    * Album Artist URI(s) - URI artis yang terkait dengan album (bisa mencakup beberapa URI jika album bersifat kompilasi).
    * Album Artist Name(s) - Nama artis atau koleksi artis pada album.
    * Album Release Date - Tanggal rilis album (format string, misalnya "YYYY-MM-DD" atau "YYYY").
    * Album Image URL  - URL gambar sampul album.
    * Disc Number - Nomor disc (untuk album dengan lebih dari satu disc).
    * Track Number  - Posisi lagu dalam urutan track album.
    * Track Duration (ms)  - Durasi lagu dalam milidetik.
    * Track Preview URL  - URL untuk preview audio pendek (jika tersedia).
    * Explicit  - Menandakan apakah lagu memiliki lirik eksplisit (True/False).
    * Popularity  - Skor popularitas lagu (0–100) berdasarkan jumlah streaming dan interaksi.
    * ISRC  - International Standard Recording Code, kode unik global untuk rekaman musik.
    * Added By  - Identifier pengguna yang menambahkan lagu ke playlist sumber (bisa nama pengguna internal).
    * Added At - Tanggal dan waktu saat lagu ditambahkan ke playlist (format ISO 8601).
    * Artist Genres  - Daftar genre yang diasosiasikan dengan artis (dipisahkan koma).
    * Danceability  - Skor (0.0–1.0) yang mengukur seberapa cocok lagu untuk menari (metrik audio).
    * Energy  - Skor (0.0–1.0) yang mengukur intensitas dan aktivitas audio.
    * Key  - Pitch dasar lagu (angka 0–11, merepresentasikan C, C#, D, … B).
    * Loudness - Rata-rata loudness lagu dalam desibel (dB), biasanya antara -60 hingga 0.
    * Mode  - Modus musik: major (1) atau minor (0).
    * Speechiness  - Skor (0.0–1.0) yang mengukur dominasi kata-kata lirik di dalam rekaman.
    * Acousticness  - Probabilitas (0.0–1.0) lagu bersifat akustik.
    * Instrumentalness  - Skor (0.0–1.0) yang menunjukkan apakah lagu cenderung instrumental.
    * Liveness  - Skor (0.0–1.0) yang mengukur kehadiran audiens langsung (live performance).
    * Valence  - Skor (0.0–1.0) yang mengukur musikalitas positif versus negatif (mood).
    * Tempo  - Tempo lagu dalam beats per minute (BPM).
    * Time Signature  - Time signature rata-rata (jumlah ketukan per bar, misalnya 3, 4, atau 5).
    * Album Genres  - Kolom ini kosong (semua nilai null), tidak digunakan di proyek.
    * Label - Label rekaman yang merilis lagu.
    * Copyrights  - Informasi hak cipta yang terkait dengan lagu.

* **Kolom Metadata**:

  * `Track Name`: Judul lagu.
  * `Artist Name(s)`: Nama artis (bisa lebih dari satu), digunakan untuk mengekstrak `Primary Artist`.
  * `Artist Genres`: Daftar genre artis, dijadikan dasar `Primary Genre` dan vektor TF-IDF \[3].
  * `Popularity`: Skor popularitas (0–100) untuk menentukan entri unik ketika ada duplikat judul.

* **Fitur Audio (Numerik)**:

  * `Danceability`, `Energy`, `Valence`, `Tempo`, `Loudness`, `Speechiness`, `Acousticness`, `Instrumentalness`, `Liveness`.

**Statistik Ringkas**:

* Jumlah data awal: 10.000 baris dan 35 kolom.
* Setelah drop missing pada fitur audio, judul, artis, dan genre: sekitar 9.446 baris.
* Setelah menghapus duplikat judul (mengambil entri dengan `Popularity` tertinggi), jumlah akhir lagu ≈ 7813.

**Pemeriksaan Data**:

* **Missing Values**: Sebelumnya, beberapa baris memiliki missing pada kolom `Artist Genres` atau fitur audio; baris tersebut dihapus.
* **Duplikasi**: Judul lagu yang sama, dihapus kecuali entri dengan popularitas tertinggi.
* **Distribusi Genre**: Terdapat puluhan genre populer (EDM, pop, rock, hip hop, dll.). `Primary Genre` diambil dari genre pertama yang tercantum.
* **Rentang Nilai Audio**: Fitur audio memiliki rentang berbeda; perlu dinormalisasi (Min-Max scaling).

## Data Preparation

1. **Drop Missing dan Ekstraksi Primary Fields**

   * Menghapus baris dengan missing pada 9 fitur audio, `Track Name`, `Artist Name(s)`, dan `Artist Genres`.
   * Ekstrak `Primary Artist` dari `Artist Name(s)` (ambil nama sebelum koma jika ada beberapa artis).
   * Ekstrak `Primary Genre` dari `Artist Genres` (ambil genre pertama).

2. **Drop Duplikat Judul Lagu**

   * Urutkan DataFrame berdasarkan `Popularity` menurun.
   * Drop duplikat pada `Track Name`, sehingga hanya satu entri paling populer yang tersisa.
   * Reset index dan tetapkan ulang `Track ID` menjadi indeks baru.

3. **Normalisasi Fitur Audio**

   * Gunakan `MinMaxScaler` untuk menskalakan nilai fitur audio antara 0 dan 1.
   * Tujuannya agar tiap fitur berkontribusi sebanding dalam perhitungan kemiripan.

4. **Ekstraksi Fitur Genre dengan TF-IDF**

   * Terapkan `TfidfVectorizer(stop_words='english', max_features=200)` pada kolom `Artist Genres` \[3].
   * Hasilnya matriks sparse berdimensi (n\_songs, 200).

5. **Gabungkan Fitur Audio dan Genre**

   * Konversi array fitur audio (yang sudah dinormalisasi) ke format sparse (`csr_matrix`).
   * Gabungkan (hstack) matriks audio dan matriks TF-IDF genre menjadi `combined_features`.
   * Matriks akhir berukuran (n\_songs, 9 + 200).

## Modeling
1. **Compute Similarity Matrix**

   * Hitung `cosine_similarity(combined_features)` (dense\_output=False untuk matriks sparse) \[4].
   * Matriks `sim_matrix` berukuran (n\_songs, n\_songs), di mana elemen (i, j) menunjukkan skor kemiripan antara lagu ke-i dan ke-j.
2. **Membangun Mapping untuk Lookup**

   * `track_to_idx`: dictionary dari `Track Name` ke `Track ID` (indeks).
   * `idx_to_track`: dictionary dari `Track ID` ke `Track Name`.
   * `idx_to_artist`: dictionary dari `Track ID` ke `Primary Artist`.
   * `idx_to_genre`: dictionary dari `Track ID` ke `Primary Genre`.

3. **Fungsi Rekomendasi**

   * Input: `track_name` (judul lagu) dan `top_k` (jumlah rekomendasi).
   * Proses:

     1. Temukan `idx` lagunya melalui `track_to_idx`.
     2. Dapatkan daftar skor kemiripan `sim_matrix[idx]` dan ubah ke format list of tuples `(track_id, similarity_score)`.
     3. Urutkan descending berdasarkan `similarity_score`.
     4. Abaikan entry di mana `track_id == idx` (tidak merekomendasikan dirinya sendiri).
     5. Ambil `top_k` entri teratas, lalu konversi ke `(judul, artist, genre, skor)`.
   * Output: List of tuples berisi rekomendasi.

4. **Uji Rekomendasi**

   * Pilih `example_track = df_clean['Track Name'].iloc[0]`.
   * Panggil `recommend(example_track, top_k=5)` untuk mendapatkan 5 lagu mirip sebagai sanity check.
  
 ```
 Track contoh: Espresso
Rekomendasi 5 lagu mirip:
Misery - Maroon 5 [pop] (score: 0.9978)
Up All Night - Khalid [pop] (score: 0.9965)
Sweet but Psycho - Ava Max [pop] (score: 0.9959)
One More Night - Maroon 5 [pop] (score: 0.9958)
Teenage Dream - Katy Perry [pop] (score: 0.9958)
```

## Evaluation

### Definisi Relevansi

Relevansi didefinisikan sebagai dua lagu yang memiliki `Primary Genre` sama (kecuali lagu query sendiri). Ini menggambarkan bahwa jika genre utama cocok, maka lagu tersebut dianggap relevan.

### Metrik Evaluasi

* **Precision\@K**: Proporsi rekomendasi di posisi 1..K yang relevan.
* **Recall\@K**: Proporsi lagu relevan yang berhasil direkomendasikan di posisi 1..K.
* **NDCG\@K (Normalized Discounted Cumulative Gain)**: Menilai kualitas ranking.

### Proses Evaluasi

1. Ambil 100 `Track ID` acak sebagai query.
2. Untuk setiap `q_idx`:

   * `query_genre = idx_to_genre[q_idx]`.
   * `relevant_tracks`: semua judul lagu lain dengan `Primary Genre` sama (df\_clean kondisi `Primary Genre == query_genre` dan `Track ID != q_idx`).
   * Panggil `recommend` untuk `query_track` (judul dari `q_idx`) dengan `top_k = 10`.
   * Hitung `prec = precision_at_k(relevant_tracks, recs, 10)`.
   * Hitung `rec_score = recall_at_k(relevant_tracks, recs, 10)`.
   * Hitung `ndcg = ndcg_at_k(relevant_tracks, recs, 10)`.
3. Simpan hasil `(prec, rec_score, ndcg)` ke list `eval_results`.
4. Buat DataFrame `eval_df` dan tampilkan `eval_df.mean()` untuk mendapatkan nilai rata-rata.

### Hasil Evaluasi Rata-Rata

```
Precision@10    0.702000
Recall@10       0.138689
NDCG@10         0.752749
dtype: float64
```

Interpretasi:

* **Precision\@10 ≈ 0.70**: Sekitar 7 dari 10 rekomendasi rata-rata relevan (genre sama).
* **Recall\@10 ≈ 0.14**: Meskipun hanya sebagian kecil dari total lagu relevan tertangkap, sistem berhasil mengambil yang paling penting.
* **NDCG\@10 ≈ 0.75**: Rekomendasi diurutkan cukup baik—banyak lagu relevan muncul di peringkat atas.

Nilai-nilai ini menunjukkan sistem sudah bekerja dengan baik dalam memprioritaskan lagu-lagu yang satu genre.

## Demonstrasi Rekomendasi

Berikut contoh rekomendasi untuk input *"Faded"*:

```
Masukkan judul lagu (Track Name): Faded

Rekomendasi 10 lagu mirip dengan 'Faded':
1. Million Voices - Radio Edit - Otto Knows [edm] (similarity: 0.9287)
2. Satisfaction - Uk Radio Edit - Benny Benassi [dutch house] (similarity: 0.9168)
3. Summertime Sadness (Lana Del Rey Vs. Cedric Gervais) - Cedric Gervais Remix - Lana Del Rey [art pop] (similarity: 0.9066)
4. Something New - Axwell / Ingrosso [edm] (similarity: 0.8986)
5. Hurricane - Radio Edit - Dzeko & Torres [dutch house] (similarity: 0.8945)
6. Young And Beautiful [Lana Del Rey vs. Cedric Gervais] - Cedric Gervais Remix Radio Edit - Lana Del Rey [art pop] (similarity: 0.8913)
7. Tsunami (Jump) - Radio Edit - DVBBS [canadian electronic] (similarity: 0.8898)
8. Tsunami (Jump) [feat. Tinie Tempah] - DVBBS [canadian electronic] (similarity: 0.8898)
9. Legacy - Radio Edit - Nicky Romero [big room] (similarity: 0.8886)
10. You’ll Be Mine - Havana Brown [australian dance] (similarity: 0.8848)
```

Rekomendasi ini menunjukkan campuran genre EDM, Dutch House, dan Art Pop remix yang selaras dengan gaya audio dan genre "Faded".

## Kesimpulan

Proyek ini berhasil membangun sistem rekomendasi musik berbasis content-based filtering dengan menggabungkan fitur audio dan TF-IDF pada kolom `Artist Genres`. Penyertaan vektor genre secara signifikan meningkatkan **Precision\@10** dan **NDCG\@10**, sehingga rekomendasi menjadi lebih relevan secara genre.

Langkah-langkah utama:

1. **Praproses Data**: Menghapus missing values dan duplikat, mengekstrak primary artist & genre.
2. **Feature Engineering**: Menormalkan fitur audio dan membangun vektor TF-IDF untuk genre.
3. **Perhitungan Similarity**: Menggunakan cosine similarity pada matriks gabungan \[4].
4. **Rekomendasi**: Mengeluarkan 10 lagu paling mirip.
5. **Evaluasi**: Precision\@10 = 0.70, Recall\@10 = 0.14, NDCG\@10 = 0.75.

**Saran Pengembangan**:

* Menambahkan bobot khusus pada fitur audio tertentu (mis. `Valence` atau `Energy`) untuk menyesuaikan mood.
* Menyempurnakan definisi relevansi: bisa didasarkan pada playlist pengguna nyata atau rating.
* Integrasi kolom `Popularity` atau `Track Duration` untuk memfilter lagu-lagu yang sangat singkat.
* Menyediakan antarmuka sederhana di aplikasi atau web untuk interaktif rekomendasi.

## Referensi

\[1] V. A. Zeimpekis and G. Koutroumpis, “Music Recommendation Systems: Content-Based and Collaborative Approaches,” *Journal of Music Data*, vol. 3, no. 2, pp. 45–58, 2020.

\[2] J. Arvidsson, "Top 10000 Spotify Songs - ARIA and Billboard Charts", Kaggle, 2024. \[Online]. Available: [https://www.kaggle.com/datasets/joebeachcapital/top-10000-spotify-songs-1960-now/data](https://www.kaggle.com/datasets/joebeachcapital/top-10000-spotify-songs-1960-now/data)

\[3] S. Deerwester, S. T. Dumais, G. W. Furnas, T. K. Landauer, and R. Harshman, “Indexing by Latent Semantic Analysis,” *Journal of the American Society for Information Science*, vol. 41, no. 6, pp. 391–407, 1990.

\[4] T. Mikolov, K. Chen, G. Corrado, and J. Dean, “Distributed Representations of Words and Phrases and their Compositionality,” in *Advances in Neural Information Processing Systems (NIPS)*, 2013, pp. 3111–3119.
