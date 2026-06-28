# 🧠 Stance Detection pada Komentar Instagram tentang Isu Politik Indonesia
> Klasifikasi opini publik (Pro / Kontra / Netral) menggunakan IndoBERT + Self-Training Semi-Supervised Learning

---

## 🎯 Latar Belakang

Media sosial seperti Instagram menjadi ruang utama masyarakat Indonesia dalam mengekspresikan pendapat terhadap isu politik. Namun, volume komentar yang sangat besar membuat analisis manual tidak mungkin dilakukan secara skala besar.

Proyek ini membangun sistem **stance detection** otomatis yang mampu mengklasifikasikan posisi opini setiap komentar — **Pro, Kontra, atau Netral** — terhadap isu politik yang dikaji, dengan memanfaatkan model bahasa **IndoBERT** dan teknik **Self-Training** untuk memaksimalkan pemanfaatan data yang tidak berlabel.

---

## ❓ Pertanyaan Penelitian

1. Bagaimana distribusi stance masyarakat (Pro/Kontra/Netral) terhadap isu politik yang dikaji?
2. Seberapa efektif IndoBERT untuk klasifikasi stance pada teks informal Bahasa Indonesia?
3. Apakah Self-Training dapat meningkatkan performa model dengan memanfaatkan data tidak berlabel?
4. Bagaimana dinamika opini publik berubah dari waktu ke waktu?

---

## 📁 Dataset

| Detail | Keterangan |
|---|---|
| **Sumber** | Scraping mandiri via Instagram API (instagrapi) |
| **Jumlah** | 8.000+ komentar Instagram |
| **Topik** | Isu politik Indonesia |
| **Postingan** | 15 Reel & Post Instagram |
| **Label** | Pro / Kontra / Netral |
| **Anotator** | 3 anotator independen |

### Pipeline Pengumpulan Data
```
Instagram Reel/Post
        │
        ▼
Scraping komentar induk (instagrapi)
        │
        ▼
Filter otomatis (hapus spam, iklan, mention-only)
        │
        ▼
Sampling 15% → Data Anotasi (manual labeling)
        │
        ├── Data Berlabel   → Training + Testing
        └── Data Sisa       → Pool Self-Training
```

---

## 🔄 Alur Metodologi

```
Data Komentar Mentah
        │
        ▼
Preprocessing Teks
(normalisasi alay, emoji → teks, hapus mention/URL/hashtag)
        │
        ▼
Anotasi Manual (3 Anotator)
(uji reliabilitas: Fleiss Kappa)
        │
        ▼
EDA & Analisis Distribusi Label
        │
        ▼
IndoBERT Fine-tuning — Teacher Model (Gen 0)
(10-Fold Stratified Cross Validation)
        │
        ▼
Self-Training Loop
(prediksi pool → seleksi pseudo-label confidence ≥ 0.80)
(retrain → Student Model Gen 1, 2, ...)
        │
        ▼
Evaluasi: Teacher vs Student
(Accuracy, F1-Macro, Paired T-Test, Wilcoxon)
        │
        ▼
Evaluasi Data Testing
(Confusion Matrix, ROC Curve, Classification Report)
        │
        ▼
Analisis Dinamika Opini Publik
(tren per minggu: Pro / Kontra / Netral)
```

---

## 🛠️ Tools & Library

| Kategori | Library / Tools |
|---|---|
| Scraping | `instagrapi` |
| Data Wrangling | `pandas`, `numpy` |
| Preprocessing | `re`, `emoji` |
| Deep Learning | `PyTorch`, `HuggingFace Transformers` |
| Model | `indobenchmark/indobert-base-p1` |
| Evaluasi | `scikit-learn`, `statsmodels` |
| Visualisasi | `matplotlib`, `seaborn`, `wordcloud` |
| Platform | Kaggle Notebooks (GPU T4) |

---

## 🤖 Arsitektur Model

### IndoBERT
Model Transformer berbasis BERT yang dilatih pada corpus Bahasa Indonesia berskala besar, mencakup Wikipedia Indonesia, berita online, dan data web lainnya.

### Self-Training (Semi-Supervised)
Teknik pembelajaran semi-supervised untuk memaksimalkan data tidak berlabel:

```
Iterasi Self-Training:
  1. Train Teacher Model pada data berlabel
  2. Prediksi seluruh pool data tidak berlabel
  3. Seleksi prediksi dengan confidence ≥ threshold (0.80 → 0.85 → 0.90)
  4. Tambahkan sebagai pseudo-label dengan Confidence-Weighted Loss
  5. Retrain → Student Model (generasi berikutnya)
```

### Confidence-Weighted Loss
Pseudo-label tidak diperlakukan sama dengan label asli. Bobot loss = confidence score model → pseudo-label yang lebih yakin mendapat pengaruh lebih besar pada training.

### MC Dropout (Uncertainty Estimation)
Model dijalankan N kali dalam mode training (dropout aktif) untuk mendapatkan distribusi prediksi — standar deviasi antar pass digunakan sebagai ukuran uncertainty. Prediksi dengan uncertainty tinggi dibuang meski confidence-nya besar.

---

## 📈 Hasil

### Performa Model

| Metrik | Teacher (Gen 0) | Student (Final) | Peningkatan |
|---|---|---|---|
| **Accuracy** | 79% | **81.8%** | 3% |
| **F1-Macro** | 0.75 | 0.78 | 0.03 |

> *Nilai lengkap tersedia setelah notebook dijalankan*

### Reliabilitas Anotasi

| Metrik | Nilai | Interpretasi |
|---|---|---|
| **Fleiss Kappa** | 0.782 | Substantial |

### Uji Statistik
- **Paired T-Test**: menguji apakah peningkatan Student atas Teacher signifikan secara statistik
- **Wilcoxon Signed-Rank**: uji non-parametrik sebagai pembanding

---

## 💡 Kontribusi & Inovasi

1. **Pipeline NLP end-to-end** untuk teks informal Bahasa Indonesia — dari scraping hingga analisis temporal
2. **Kamus normalisasi alay** yang dikembangkan secara manual untuk konteks media sosial Indonesia
3. **Confidence-Weighted Self-Training** — pseudo-label diberi bobot sesuai keyakinan model, bukan binary accept/reject
4. **MC Dropout Uncertainty** — menyaring pseudo-label berisiko tinggi (confidence tinggi tapi tidak stabil)
5. **Analisis temporal opini publik** — memvisualisasikan pergeseran stance dari waktu ke waktu

---

## 📂 Struktur File

```
nlp-stance-detection/
│
├── nlp_stance_detection.ipynb     ← Notebook utama (kode lengkap)
├── cobascrap.py                   ← Script scraping komentar Instagram
├── filtering.py                   ← Script filter & sampling data
├── praproses.py                   ← Modul preprocessing teks
└── README.md                      ← Dokumen ini

/kaggle/input/skripsi2/            ← Kaggle Dataset (upload manual)
├── Data_Anotasi.xlsx              ← Data anotasi 3 anotator
├── Data_Berlabel.xlsx             ← Data dengan label final
├── data_training.xlsx             ← Data training berlabel
├── data_training_aug.xlsx         ← Data training + augmentasi
├── data_testing.xlsx              ← Data testing
├── data_sisa_new.xlsx             ← Pool data tidak berlabel
└── best_model_final/              ← Folder model IndoBERT terbaik
```

---

## 🚀 Cara Menjalankan di Kaggle

```
1. Buat Kaggle Dataset baru bernama "skripsi2"
2. Upload semua file Excel + folder best_model_final ke dataset tersebut
3. Buat Notebook baru di Kaggle
4. Tambahkan dataset "skripsi2" ke notebook
5. Aktifkan GPU: Settings → Accelerator → GPU T4
6. Upload nlp_stance_detection.ipynb
7. Jalankan dari atas ke bawah
```

**Estimasi waktu training:** 2–4 jam (10-Fold × 2 Generasi, GPU T4)

---

## ⚠️ Catatan Etika & Privasi

- Scraping dilakukan pada komentar publik di konten publik Instagram
- Username pengguna tidak ditampilkan dalam analisis akhir
- Data digunakan semata-mata untuk keperluan penelitian akademis
- Tidak ada informasi pribadi yang dipublikasikan

---

## 📚 Referensi

- Devlin, J., Chang, M. W., Lee, K., & Toutanova, K. (2019). BERT: Pre-training of deep bidirectional transformers for language understanding.
- Mukherjee, S., & Awadallah, A. H. (2020). Uncertainty-aware Self-training for Few-shot Text Classification. NeurIPS
- Wilie, B., Vincentio, K., Winata, G. I., Cahyawijaya, S., Li, X., Lim, Z. Y., Soleman, S., Mahendra, R., Fung, P., Bahar, S., & Purwarianti, A. (2020). IndoNLU: Benchmark and Resources for Evaluating Indonesian Natural Language Understanding.

---

*Proyek ini merupakan implementasi skripsi S1 Statistika — Universitas Halu Oleo, Kendari (2025).*  
*Akurasi terbaik: 80.6% pada data testing independen.*

