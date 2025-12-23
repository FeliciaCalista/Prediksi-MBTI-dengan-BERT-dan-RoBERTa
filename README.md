# Prediksi dan Klasifikasi MBTI dengan BERT & RoBERTa

## Latar Belakang
Tipe kepribadian **Myers-Briggs Type Indicator (MBTI)** merupakan salah satu alat psikometri populer saat ini. MBTI digunakan dalam berbagai bidang, seperti konseling dan rekrutmen pekerja. Namun, pengukuran MBTI biasanya masih dilakukan secara manual melalui kuesioner dan berdasarkan kesadaran diri responden, sehingga kurang efisien dan hasil pengukuran tidak selalu mencerminkan perilaku nyata responden.  

Dengan **Natural Language Processing (NLP)**, dimungkinkan untuk menganalisis MBTI seseorang dari pola bahasa yang digunakan. Beberapa penelitian menunjukkan bahwa orang dengan tipe MBTI yang sama memiliki gaya tulisan yang serupa di media sosial. NLP dapat mengenali kata dan kalimat penting yang mencerminkan karakter pengguna.  

Referensi: [ScienceDirect](https://www.sciencedirect.com/science/article/abs/pii/S0957417425029100)

---

## Data
Dataset yang digunakan adalah **Myers-Briggs Personality Type Dataset**, diperoleh dari [Kaggle](https://www.kaggle.com/datasets/datasnaek/mbti-type).  

Dataset terdiri dari 8.675 sampel dengan dua kolom utama:  

1. **type (Label)**: Kode 4 huruf yang merepresentasikan tipe kepribadian (misal: INFJ, ENTP) — target yang akan diprediksi.  
2. **posts (Fitur/Input)**: Kumpulan 50 postingan terakhir dari setiap pengguna, digabung dengan delimiter `|||`.  

### Preprocessing
- Memisahkan postingan berdasarkan delimiter atau memotong teks agar sesuai batas token BERT (512 token).  
- Membersihkan URL dan karakter khusus dari teks.  
- Menangani **class imbalance**, karena kelas mayoritas seperti INFP, INFJ, INTP, INTJ mendominasi, sedangkan kelas minoritas seperti ESTJ, ESFJ memiliki jumlah sedikit.  

---

## Algoritma

### BERT
BERT adalah model **bidirectional Transformer encoder** yang kuat dalam klasifikasi teks. Model ini telah melalui pre-training menggunakan **Masked Language Modeling (MLM)** dan **Next Sentence Prediction (NSP)**.  

#### Komponen Utama:
1. **Word Embedding**: Token, Positional, dan Segment Embedding.  
2. **Positional Encoding**: Memberikan informasi urutan kata.  
3. **Multi-Head Attention**: Melihat kata di kiri dan kanan secara bersamaan.  
4. **Feed-Forward Network**: Digunakan selama pre-training untuk self-supervised learning.  

#### Fine-Tuning:
- Layer output diganti untuk klasifikasi MBTI (16 kelas).  
- Supervised learning dengan dataset MBTI.  

#### Kelebihan BERT:
- Pre-trained model: menangkap konteks dan nuansa bahasa.  
- Fine-tuning mudah dengan `BertForSequenceClassification`.  
- Cocok untuk klasifikasi multi-kelas (16 tipe MBTI).  

### RoBERTa
RoBERTa adalah model pembanding berbasis BERT dengan perbedaan pada pretraining (menghapus NSP, dynamic masking, dan dataset lebih besar).  

| Aspek                     | BERT                                   | RoBERTa                                      |
|----------------------------|----------------------------------------|----------------------------------------------|
| Arsitektur dasar           | Transformer Encoder                    | Transformer Encoder (sama seperti BERT)      |
| Pretraining objective      | MLM + NSP                               | MLM saja                                     |
| Masking strategy           | Static masking                          | Dynamic masking                              |
| Next Sentence Prediction   | Digunakan                               | Tidak digunakan                              |
| Data pelatihan             | Wikipedia + BookCorpus (~3B kata)       | Dataset lebih besar (~160GB teks)            |
| Batch size                 | Relatif lebih kecil                     | Lebih besar                                  |
| Optimasi training          | Standar                                 | Lebih agresif & teroptimasi                  |
| Performa downstream task   | Baik                                    | Umumnya lebih baik dari BERT                 |
| Kebutuhan komputasi        | Lebih ringan                            | Lebih berat                                  |

---

## Tokenization
- `posts` → tokenized menjadi `input_ids` dan `attention_mask`.  
- Padding untuk menyamakan panjang input batch.  
- Truncation sesuai batas maksimal BERT (512 token).  

---

## Model Setup
- `compute_metrics` → menghitung **accuracy, precision, recall, F1-score**.  
- `WeightedTrainer(Trainer)` → menggunakan **class weight** agar kelas minoritas lebih diperhatikan.  
- **Class weight** memberikan bobot lebih pada kesalahan kelas minoritas untuk mengurangi bias.  

---

## Training
- Fine-tuning BERT dan RoBERTa menggunakan dataset MBTI.  
- Hyperparameter utama:  
  - Epochs: 3  
  - Batch size: 8  
  - Learning rate: BERT (2e-5), RoBERTa (5e-5)  
  - Weight decay: 0.01  

---

## Evaluasi
- Metrik evaluasi: **Accuracy, F1-Score**
  - Akurasi mengukur proporsi prediksi yang benar terhadap seluruh data. Metrik ini kurang representatif pada dataset yang tidak seimbang
  - F1-Score (Weighted) memperhitungkan proporsi setiap kelas. Setiap nilai F1 per kelas diberi bobot sesuai dengan jumlah sampe sehingga hasil evaluasi mencerminkan performa model pada seluruh dataset dan bukan hanya pada kelas mayoritas.
- Digunakan **Confusion Matrix** untuk memastikan performa di semua kelas.

---

## Pengujian
- Model diuji pada dataset baru.   

---

## Referensi

### Paper / Jurnal
- Mahdavi, A. et al. (2023). *Process of Fine-tune BERT model by transforming the feature space Z Layer*. [Link](https://www.researchgate.net/profile/Atefeh-Mahdavi-5/publication/373518572/figure/fig1/AS:11431281184804636@1693451040732/Process-of-Fine-tune-BERT-model-by-transforming-the-feature-space-Z-Layer-and-enhancing.ppm)
- Diva Portal. (2023). *Full Text Research on Transformer Models*. [Link](https://www.diva-portal.org/smash/get/diva2:1970734/FULLTEXT01.pdf)
- ECUST Journal. *Supplementary Research on NLP & Transformers*. [Link](https://journal.ecust.edu.cn/en/supplement/97a7d394-ed98-4401-b23d-b4caf43437f6)

### Blog / Artikel
- DSStream. (2023). *RoBERTa vs BERT: Exploring the Evolution of Transformer Models*. [Link](https://www.dsstream.com/post/roberta-vs-bert-exploring-the-evolution-of-transformer-models)
- McCormick, C. (2019). *BERT Fine-Tuning Guide*. [Link](https://mccormickml.com/2019/07/22/BERT-fine-tuning/)
- Achimoraites. (2022). *Fine-tuning RoBERTa for Topic Classification*. [Link](https://achimoraites.medium.com/fine-tuning-roberta-for-topic-classification-with-hugging-face-transformers-and-datasets-library-c6f8432d0820)

### Dataset
- Tonylica. (2022). *MBTI: A NLP Model Approach*. [Kaggle](https://www.kaggle.com/code/tonylica/mbti-a-nlp-model-approach)

### Video / Tutorial
- YouTube. (2023). *Fine-Tuning BERT for NLP*. [Link](https://www.youtube.com/watch?v=GDN649X_acE)
- YouTube. (2023). *RoBERTa vs BERT Explanation*. [Link](https://youtu.be/xI0HHN5XKDo?si=7n2nUbJJ1-mfNzTv)
- YouTube. (2023). *Transformers for NLP Overview*. [Link](https://youtu.be/4QHg8Ix8WWQ?si=vggzGHfhEkjU4_lu)

### Gambar / Visualisasi
- Bukittinggi MBTI Types. [Link](https://cdn.rri.co.id/berita/Bukittinggi/o/1719477122444-mbti-typesjpg-20220217032149/sd5zza9sghwmsis.jpeg)
- BERT Fine-Tuning Process Diagram. [Link](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRbq390DtGQyAS07weYKglje8JXzHltcxJYrQ&s)
