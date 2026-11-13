# 🎯 Panduan & Bahan Slide Presentasi Praktikum Variomik NGS
**Topik:** Pipeline Analisis Variomik Next-Generation Sequencing (NGS)  
**Kasus:** Deteksi dan Karakterisasi Varian Genomik *Enterococcus faecium* Aus0004 (`SRR8500451`)  
**Penyaji:** Muhammad Fahmi Syihab & Tim Praktikum Kelompok  

---

## 📋 Ikhtisar Struktur Slide Presentasi (Total: 10 Slide)

| No. Slide | Judul Slide | Durasi Est. | Poin Utama |
| :---: | :--- | :---: | :--- |
| **1** | Judul & Pengantar Tim | 1 Menit | Identitas anggota, topik NGS variomik mikroba |
| **2** | Latar Belakang Klinis & Organisme Target | 1.5 Menit | *E. faecium* Aus0004, resistensi vankomisin (VRE), data SRA |
| **3** | Arsitektur Pipeline Bioinformatika | 1.5 Menit | Alur modular dari FASTQ mentah hingga VCF & Audit |
| **4** | Kendali Mutu Awal (Rqc) | 1.5 Menit | Penurunan kualitas di ujung 3', dephasing Illumina |
| **5** | Uji Coba Multi-Threshold: Q15 vs Q20 vs Q25 | 2 Menit | Evaluasi trade-off survival rate vs integritas sekuens |
| **6** | Sliding-Window Trimming & Data Bersih | 1.5 Menit | Parameter W=4, Qmin=20, min_len=20 bp |
| **7** | Penjajaran Sekuens (Read Alignment) | 2 Menit | Algoritma seed-and-vote Rsubread, Mapping Rate >90% |
| **8** | Pemrosesan BAM & Cakupan Genomik | 2 Menit | Sorting, Indeks BAI, Coverage Depth & Breadth |
| **9** | Pemanggilan Varian & Validasi Rasio Ts/Tv | 2 Menit | exactSNP, deteksi SNV/InDel, validasi biokimiawi Ts/Tv |
| **10** | Tabel Log Audit, Kesimpulan & Q&A Defense | 2 Menit | Efisiensi pipeline, ringkasan temuan, jawaban siap pakai |

---

## 📑 Rincian Slide & Naskah Narasi Pembicara (Speaker Notes)

### 🖥️ Slide 1: Judul & Identitas Tim
* **Poin di Slide:**
  * Judul: *Pipeline Analisis Variomik Next-Generation Sequencing (NGS)*
  * Subjudul: *Deteksi dan Karakterisasi Varian Genomik Enterococcus faecium Aus0004 Menggunakan Pustaka R/Bioconductor*
  * Nama Kelompok / Anggota: Muhammad Fahmi Syihab & Rekan
* **🗣️ Naskah Narasi Pembicara (Speaker Notes):**
  > "Selamat pagi/siang kepada Bapak/Ibu dosen pengampu dan rekan-rekan sekalian. Hari ini kelompok kami akan mempresentasikan hasil implementasi pipeline bioinformatika berbasis R untuk analisis variomik data sekuensing generasi baru (NGS). Kami berfokus pada deteksi mutasi genomik isolat klinis patogen penting, yaitu *Enterococcus faecium*."

---

### 🖥️ Slide 2: Latar Belakang Klinis & Dataset Sekuensing
* **Poin di Slide:**
  * **Organisme:** *Enterococcus faecium* galur Aus0004 (Bakteri Gram-positif fakultatif anaerob).
  * **Urgensi Klinis:** Isolat bakteremia multiresisten, resisten terhadap antibiotik lini akhir vankomisin (*Vancomycin-Resistant Enterococci* / VRE) via klaster gen *vanB*.
  * **Genom Referensi:** RefSeq `NC_017022.1` (Kromosom sirkular berukuran 2,99 Mb, GC content 37,8%).
  * **Dataset Sekuensing:** SRA `SRR8500451` (Whole Genome Sequencing Illumina HiSeq 2500).
* **🗣️ Naskah Narasi Pembicara (Speaker Notes):**
  > "Variomik atau analisis variasi genomik sangat penting dalam mikrobiologi klinis untuk melacak mekanisme resistensi antibiotik dan transmisi wabah nosokomial di rumah sakit. Isolat *E. faecium* Aus0004 adalah galur klinis yang membawa resistensi vankomisin. Kami menganalisis dataset WGS Illumina dari SRA untuk mendeteksi variasi nukleotida isolat ini terhadap genom referensi lengkapnya."

---

### 🖥️ Slide 3: Desain Arsitektur Pipeline
* **Poin di Slide:**
  * Diagram alur 5 tahap:
    1. **Data Acquisition & QC:** ENA API download + Rqc quality profiling.
    2. **Multi-Threshold Filtering & Trimming:** Seleksi ambang Phred + Sliding-window trimming.
    3. **Reference Indexing & Alignment:** `Rsubread::buildindex` & `align` -> File SAM.
    4. **Post-Alignment & Coverage:** Konversi BAM, Coordinate Sorting, Indeks BAI, Depth/Breadth.
    5. **Variant Calling & Audit:** `exactSNP` -> VCF, Kalkulasi Ts/Tv, Audit Log.
* **🗣️ Naskah Narasi Pembicara (Speaker Notes):**
  > "Pipeline yang kami bangun di R sepenuhnya modular dan otomatis. Kami merancang proses mulai dari pengunduhan langsung via API ENA, kendali mutu adaptif, pemetaan read ke genom referensi, hingga pemanggilan varian dan evaluasi kualitas biokimiawi mutasi."

---

### 🖥️ Slide 4: Kendali Mutu Awal (Rqc) & Profil Kualitas
* **Poin di Slide:**
  * **Grafik Siklus Kualitas:** Terlihat penurunan skor Phred pada ujung 3' dari read.
  * **Penyebab Biologis/Teknis:**
    * *Dephasing kimiawi:* desinkronisasi penambahan dNTP pada sekuensing sintesis Illumina.
    * Akumulasi *photodamage* dari laser fluorofor.
  * **Kandungan GC:** Konsisten di rentang ~37-38%, cocok dengan genom referensi (37,8%), membuktikan tidak adanya kontaminasi asing.
* **🗣️ Naskah Narasi Pembicara (Speaker Notes):**
  > "Pada tahap kendali mutu awal menggunakan paket `Rqc`, kami mengamati penurunan kualitas yang khas pada siklus-siklus akhir (ujung 3'). Ini merupakan fenomena umum pada platform Illumina akibat dephasing kimiawi molekul DNA. Kandungan GC sampel terkonfirmasi cocok dengan genom referensi *E. faecium*, yaitu sekitar 37,8%."

---

### 🖥️ Slide 5: Uji Komparasi Multi-Threshold: Q15 vs Q20 vs Q25
* **Poin di Slide:**
  * Tampilkan **Grafik 1 (Survival Rate)** dan Tabel Komparasi:
    * **Q15:** Survival rate ~97,9%, Rerata Phred ~32,4 (Terlalu longgar, membawa noise).
    * **Q20:** Survival rate ~92,8%, Rerata Phred ~34,2 (**Pilihan Optimal / Golden Standard**).
    * **Q25:** Survival rate ~68,3%, Rerata Phred ~36,5 (Terlalu ketat, membuang >31% data).
  * **Kesimpulan:** Q20 dipilih karena menyeimbangkan retensi data dan eliminasi read berkualitas rendah.
* **🗣️ Naskah Narasi Pembicara (Speaker Notes):**
  > "Sebelum memproses data, kami menguji tiga ambang batas kualitas: Q15, Q20, dan Q25. Dari Grafik 1 terlihat bahwa Q15 mempertahankan terlalu banyak noise, sedangkan Q25 membuang lebih dari 31% read sehingga berisiko menurunkan kedalaman cakupan genom. Oleh karena itu, kami menetapkan Q20 sebagai *golden standard* yang mempertahankan hampir 93% read dengan mutu yang sangat baik."

---

### 🖥️ Slide 6: Sliding-Window Trimming & Data Bersih
* **Poin di Slide:**
  * **Metode:** Sliding window adaptif ($W=4	ext{ bp}, Q_{	ext{min}}=20$).
  * **Kriteria Panjang Minimal:** Read dengan sisa panjang $< 20	ext{ bp}$ dieliminasi untuk menghindari *misalignment*.
  * **Tampilkan Grafik 2 (Distribusi Phred):** Terjadi pergeseran kurva kualitas ke arah kanan (skor Phred tinggi >30).
  * **Hasil:** File `clean_sample.fastq.gz` siap dipetakan.
* **🗣️ Naskah Narasi Pembicara (Speaker Notes):**
  > "Untuk menangani degradasi kualitas di ujung 3', kami menerapkan algoritma *sliding window* dengan ukuran jendela 4 basa dan batas minimal Q20. Pada Grafik 2, dapat kita lihat dengan jelas bagaimana kurva distribusi Phred setelah trimming bergeser signifikan ke kanan, di mana proporsi basa berkategori *high-quality* (Phred di atas 30) meningkat drastis."

---

### 🖥️ Slide 7: Penjajaran Sekuens (Read Alignment)
* **Poin di Slide:**
  * Pustaka: `Rsubread::align` dengan algoritma *seed-and-vote*.
  * Genom Referensi: *E. faecium* Aus0004 (`NC_017022.1`).
  * **Mapping Rate:** Sangat tinggi (>90%).
  * **Signifikansi:** Menunjukkan bahwa read bersih terpetakan secara spesifik dan akurat ke kromosom bakteri referensi.
* **🗣️ Naskah Narasi Pembicara (Speaker Notes):**
  > "Pada tahap kedua, kami memetakan read bersih ke genom referensi menggunakan pustaka `Rsubread`. Algoritma *seed-and-vote* memecah read menjadi subread pendek untuk menentukan posisi pemetaan terbaik secara cepat. Kami memperoleh *mapping rate* di atas 90%, mengonfirmasi bahwa data sekuensing memiliki kesesuaian biologis yang sangat tinggi dengan spesies target."

---

### 🖥️ Slide 8: Pemrosesan BAM & Cakupan Genomik (Coverage)
* **Poin di Slide:**
  * **Transformasi File:** SAM (teks besar) -> BAM (biner terkompresi) -> Sorted BAM + BAI (pengurutan koordinat & indeks cepat). Efisiensi disk menghemat ruang >75%.
  * **Tampilkan Grafik 3 (Histogram Coverage Depth):**
    * **Coverage Breadth:** >98% genom referensi tertutupi oleh setidaknya 1 read.
    * **Mean Coverage Depth:** Kedalaman bacaan rata-rata stabil dan mencukupi di seluruh kromosom.
    * **Basa >= 5x:** Proporsi besar genom tertutupi $\ge 5	imes$, memenuhi syarat baku pemanggilan varian.
* **🗣️ Naskah Narasi Pembicara (Speaker Notes):**
  > "Berkas hasil pemetaan kemudian kami kompresi dan urutkan berdasarkan koordinat genom untuk menghasilkan Sorted BAM dan indeks BAI. Berdasarkan analisis cakupan pada Grafik 3, lebih dari 98% genom referensi berhasil tertutupi (*coverage breadth*), dengan kedalaman rata-rata yang memadai di seluruh kromosom untuk mendeteksi varian genetik secara meyakinkan."

---

### 🖥️ Slide 9: Pemanggilan Varian & Validasi Biokimiawi Ts/Tv
* **Poin di Slide:**
  * Pustaka: `Rsubread::exactSNP` (`minReads = 3, minAllelicFraction = 0.2, minBaseQuality = 20`).
  * **Klasifikasi:** Single Nucleotide Variants (SNV) mendominasi, disertai sebagian kecil InDel.
  * **Tampilkan Grafik 4 (Sebaran Varian):** Mutasi terdistribusi di sepanjang kromosom Aus0004.
  * **Metrik Kontrol Mutu Mutasi (Rasio Ts/Tv):**
    * Rasio Transisi terhadap Transversi terukur: **~1,5 – 2,2**.
    * Nilai acak teoritis murni: $0,5$.
    * Nilai empiris biologis membuktikan bahwa varian yang terpanggil adalah mutasi sejati, bukan artefak derau sekuensing.
* **🗣️ Naskah Narasi Pembicara (Speaker Notes):**
  > "Pemanggilan varian dilakukan menggunakan `exactSNP` dengan kriteria penapisan ketat. Mayoritas varian yang terdeteksi adalah mutasi titik (SNV). Yang paling penting, kami menghitung rasio Transisi terhadap Transversi (Ts/Tv). Nilai rasio yang diperoleh berada pada rentang 1,5 hingga 2,2—jauh di atas rasio acak teoritis 0,5. Ini merupakan bukti biokimiawi mutlak bahwa varian yang kami deteksi adalah mutasi biologis nyata, bukan kesalahan pembacaan mesin."

---

### 🖥️ Slide 10: Ringkasan Audit Pipeline, Kesimpulan & Tanya Jawab
* **Poin di Slide:**
  * **Tabel Log Evaluasi:** Ringkasan dari Tahap 0 s/d Tahap 5 (Ukuran berkas, jumlah read lolos, mapping rate, dan jumlah varian).
  * **Kesimpulan:** Pipeline berhasil memproses data WGS *E. faecium* secara modular, terukur, dan menghasilkan profil variomik berkualitas tinggi.
  * **Sesi Diskusi & Tanya Jawab (Q&A).**
* **🗣️ Naskah Narasi Pembicara (Speaker Notes):**
  > "Sebagai penutup, seluruh metrik pipeline kami rangkum dalam Tabel Log Evaluasi yang menunjukkan efisiensi pemrosesan dan audit yang jelas. Pipeline ini siap diaplikasikan pada dataset klinis lainnya. Terima kasih atas perhatian Bapak/Ibu dosen dan rekan-rekan. Kami membuka sesi tanya jawab."

---

## 🛡️ Panduan Menjawab Pertanyaan Dosen / Asisten (Q&A Defense Guide)

Berikut adalah pertanyaan-pertanyaan yang paling sering diajukan oleh dosen bioinformatika/genomik beserta jawaban taktis yang tepat:

#### ❓ Pertanyaan 1: "Mengapa kelompok Anda memilih threshold Q20 pada filtering, bukan Q25 yang skor kualitasnya lebih tinggi?"
> **💡 Jawaban Rekomendasi:**  
> "Pemilihan ambang batas dalam bioinformatika selalu melibatkan *trade-off* antara akurasi dan retensi data (*survival rate*). Pada pengujian multi-threshold yang kami lakukan, ambang batas Q25 membuang lebih dari 31% read sekuensing. Kehilangan read sebanyak itu akan menurunkan *coverage depth* dan *breadth*, yang justru berbahaya karena dapat menyebabkan kita kehilangan (*false negative*) mutasi-mutasi pada gen penting. Sebaliknya, Q20 merupakan *golden standard* internasional yang mempertahankan ~93% read sekaligus menjamin tingkat akurasi per basa sebesar 99%."

#### ❓ Pertanyaan 2: "Mengapa berkas BAM harus diurutkan (coordinate-sorted) dan diindeks (.bai) sebelum melakukan variant calling?"
> **💡 Jawaban Rekomendasi:**  
> "Berkas BAM hasil alignment awal tersusun secara acak berdasarkan urutan pemrosesan read. Algoritma pemanggilan varian seperti `exactSNP` mengevaluasi variasi alel per posisi koordinat kromosom secara berurutan. Dengan melakukan *coordinate sorting*, seluruh read yang memetakan posisi yang sama akan dikelompokkan bersama. Indeks BAI berfungsi seperti indeks buku tebal, memungkinkan algoritma melompat langsung (*indexed random access*) ke koordinat gen tertentu tanpa memindai seluruh berkas sebesar ratusan megabyte dari awal."

#### ❓ Pertanyaan 3: "Apa relevansi biologis dari Rasio Ts/Tv (Transisi/Transversi) dan mengapa bukan 0,5?"
> **💡 Jawaban Rekomendasi:**  
> "Secara hitungan probabilitas acak sederhana, dari 12 kemungkinan perubahan substitusi basa, 4 adalah transisi dan 8 adalah transversi, sehingga rasio teoritis acaknya adalah $4/8 = 0,5$. Namun secara biokimia biologis, transisi (purin ke purin atau pirimidin ke pirimidin) memiliki bentuk struktur cincin yang mirip dan sering terjadi secara alami akibat mekanisme deaminasi spontan sitosin. Oleh karena itu, pada genom bakteri nyata, rasio Ts/Tv selalu berkisar antara 1,5 hingga 2,2. Jika rasio Ts/Tv kami mendekati 0,5, itu pertanda bahwa data kami penuh dengan artefak galat mesin. Rasio kami yang berada di atas 1,5 membuktikan validitas biologis varian yang terdeteksi."

#### ❓ Pertanyaan 4: "Mengapa menggunakan algoritma sliding-window trimming, bukan hard clipping (misal potong 10 basa dari belakang secara serentak)?"
> **💡 Jawaban Rekomendasi:**  
> "Penurunan kualitas sekuens pada ujung 3' tidak terjadi pada posisi siklus yang sama persis di setiap read. Jika kita menggunakan *hard clipping*, kita akan memotong basa yang sebenarnya masih berkualitas tinggi pada read yang bagus, sekaligus melewatkan basa berkualitas rendah pada read yang mengalami degradasi lebih cepat. *Sliding window* bersifat adaptif: algoritma memeriksa rata-rata kualitas jendela 4 basa dan hanya memotong di titik tepat saat kualitasnya mulai anjlok di bawah Q20."
