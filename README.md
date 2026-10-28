# Deteksi Kematangan TBS Sawit

Repositori ini mendokumentasikan eksperimen perbandingan **YOLOv11** dan **Faster R-CNN** untuk mendeteksi tingkat kematangan tandan buah segar (TBS) sawit pada citra. Kedua model menggunakan **data, split, definisi kelas, dan bounding box yang sama** agar evaluasinya setara.

> Dokumen acuan preprocessing: **Handoff Preprocessing — Deteksi Kematangan TBS Sawit**. Aturan split dan mapping kelas di bawah ini bersifat terkunci untuk eksperimen perbandingan.

## Tujuan

Membandingkan kinerja detektor satu tahap (YOLOv11) dan dua tahap (Faster R-CNN) pada tugas deteksi objek berbasis *bounding box*, dengan pelaporan minimal:

- mAP@0.5
- mAP@0.5:0.95
- precision
- recall

Pemilihan *epoch*, *confidence threshold*, dan hiperparameter harus dilakukan pada data validasi—bukan pada test set.

## Dataset

| Item | Detail |
| --- | --- |
| Sumber | Roboflow Universe — `zah/palm-oil-fg85a`, versi 3 |
| Format unduhan awal | YOLOv11 |
| Resolusi | 640 × 640 piksel; gambar telah di-*stretch* oleh Roboflow |
| Jumlah gambar | 8.396 |
| Jumlah anotasi setelah standardisasi | 13.275 bounding box |

Dataset sumber memuat campuran anotasi deteksi (*bounding box*) dan poligon/segmentasi. Setiap poligon telah dikonversi menjadi **bounding box minimum**. Hasil standardisasi ini dipakai oleh kedua model; berkas sumber Roboflow tidak diubah.

### Kelas dan mapping ID

| Makna kelas | ID YOLOv11 | ID COCO / Faster R-CNN |
| --- | ---: | ---: |
| kurang masak | 0 | 1 |
| masak | 1 | 2 |
| mentah | 2 | 3 |
| terlalu masak | 3 | 4 |

Pada Faster R-CNN, label `0` dicadangkan untuk **background**, sehingga ID kelas objek dimulai dari `1`. Jangan mengubah urutan maupun mapping ini.

## Split data terkunci

| Split | Gambar | Bounding box |
| --- | ---: | ---: |
| Train | 5.893 | 9.266 |
| Validasi | 1.676 | 2.654 |
| Test | 827 | 1.355 |
| **Total** | **8.396** | **13.275** |

Gunakan split di atas tanpa *re-split*. Data validasi dipakai untuk pemilihan model dan hiperparameter; test set hanya dipakai untuk evaluasi akhir kedua model.

### Distribusi objek per kelas

| Kelas | Train | Validasi | Test |
| --- | ---: | ---: | ---: |
| kurang masak | 2.635 | 757 | 394 |
| masak | 1.959 | 556 | 271 |
| mentah | 2.447 | 705 | 362 |
| terlalu masak | 2.225 | 636 | 328 |

## Struktur repositori

```text
.
├── data/
│   ├── metadata/                       # Metadata dan dokumentasi data
│   └── processed/
│       ├── coco json/                  # Artefak COCO untuk Faster R-CNN
│       └── yolov11/                    # YAML dan label YOLOv11 yang dilacak Git
├── notebooks/
│   ├── fgd-dgx-preprocessing-last-final(2).ipynb
│   └── rcnn-training-dan-fine-tune-finale.ipynb
├── outputs/
│   └── faster_rcnn/                    # Keluaran eksperimen Faster R-CNN
├── weights/                             # Dokumentasi lokasi checkpoint
├── requirements.txt
└── README.md
```

## Artefak preprocessing untuk Faster R-CNN

Paket `palm_oil_preprocessing_output.zip` memuat anotasi COCO dan dokumentasi berikut. Paket ini **tidak berisi gambar asli** dan **bukan paket training YOLOv11**.

| File | Kegunaan |
| --- | --- |
| `instances_train.json` | Anotasi COCO untuk training Faster R-CNN |
| `instances_val.json` | Anotasi COCO untuk validasi Faster R-CNN |
| `instances_test.json` | Anotasi COCO untuk evaluasi Faster R-CNN |
| `class_mapping_yolo_to_coco.csv` | Referensi mapping ID antarformat |
| `image_audit.csv` | Hasil audit per gambar |
| `class_distribution_by_split.csv` | Distribusi kelas per split |
| `coco_export_summary.csv` | Ringkasan jumlah gambar dan anotasi COCO |
| `preprocessing_metadata.json` | Metadata proses dan mapping kelas |

### Struktur input Faster R-CNN

Siapkan gambar asli dan artefak JSON pada lingkungan training, misalnya Kaggle:

```text
palm-oil-images/
├── train/images/
├── valid/images/
└── test/images/

palm_oil_preprocessing_output/
├── instances_train.json
├── instances_val.json
└── instances_test.json
```

Pasangkan setiap file JSON dengan folder gambar pada split yang sesuai. Jangan gunakan label `.txt` sumber untuk Faster R-CNN.

## Struktur input YOLOv11

Repositori ini melacak YAML serta direktori label YOLOv11. Untuk training, siapkan satu dataset lengkap yang juga memuat **berkas gambar asli**, bukan *symbolic link* dari Colab:

```text
yolo_detection_ready/
├── data_yolo_detection_ready.yaml
├── train/
│   ├── images/
│   └── labels/                         # class xc yc w h
├── val/
│   ├── images/
│   └── labels/
└── test/
    ├── images/
    └── labels/
```

Sebelum training di Kaggle, sesuaikan nilai `path` pada `data_yolo_detection_ready.yaml` ke lokasi dataset Kaggle. Contoh:

```yaml
path: /kaggle/input/nama-dataset-yolo/yolo_detection_ready
train: train/images
val: val/images
test: test/images
nc: 4
names: [kurang masak, masak, mentah, terlalu masak]
```

Gunakan `data_yolo_detection_ready.yaml`, bukan `data.yaml` asli dari Roboflow.

## Menjalankan notebook

Instal dependensi proyek terlebih dahulu:

```bash
pip install -r requirements.txt
jupyter lab
```

Notebook yang tersedia:

| Notebook | Kegunaan |
| --- | --- |
| `fgd-dgx-preprocessing-last-final(2).ipynb` | Preprocessing dan eksplorasi data |
| `rcnn-training-dan-fine-tune-finale.ipynb` | Training serta *fine-tuning* Faster R-CNN |

### Menjalankan Faster R-CNN di Kaggle

Notebook Faster R-CNN ditujukan untuk Kaggle. Ia mengunduh ekspor COCO dari Roboflow ke `/kaggle/working/palm-oil-coco` dan menggunakan JSON hasil preprocessing sebagai *ground truth*. Sebelum menjalankannya:

1. Aktifkan akselerator GPU di Kaggle; notebook otomatis memakai CUDA bila tersedia dan akan memakai CPU sebagai cadangan.
2. Simpan kunci Roboflow sebagai Kaggle Secret bernama `ROBOFLOW_API_KEY`.
3. Tambahkan paket `palm_oil_preprocessing_output` sebagai Kaggle Input. Notebook mengharapkan `instances_train.json`, `instances_val.json`, dan `instances_test.json` di dalamnya.
4. Pastikan versi ekspor gambar Roboflow dan anotasi preprocessing merujuk pada split yang sama.

Notebook memvalidasi keberadaan gambar, kecocokan `file_name` pada COCO JSON, serta kategori ID 1–4 sebelum pelatihan dimulai. Kegagalan pemeriksaan tersebut berarti input Kaggle atau pasangan gambar–anotasi perlu diperbaiki.

## Implementasi Faster R-CNN

Eksperimen Faster R-CNN menggunakan `torchvision.models.detection.fasterrcnn_resnet50_fpn` dengan bobot pralatih COCO. Head prediksi diganti menjadi lima keluaran: satu kelas background internal dan empat kelas kematangan. Ukuran `min_size` dan `max_size` model ditetapkan ke 640 agar selaras dengan gambar hasil preprocessing.

| Komponen | Konfigurasi baseline |
| --- | --- |
| Seed | 42 |
| Arsitektur | Faster R-CNN ResNet-50-FPN, bobot pralatih COCO |
| Ukuran input | 640 × 640 |
| Batch size | 2 |
| Epoch maksimum | 20 |
| Optimizer | SGD |
| Learning rate | 0,005 |
| Momentum / weight decay | 0,9 / 0,0005 |
| Scheduler | StepLR, setiap 10 epoch, gamma 0,1 |
| Ambang skor evaluasi COCO | 0,05 |

Augmentasi online hanya diterapkan pada train: *horizontal flip* dengan probabilitas 0,5, serta brightness dan contrast acak pada rentang 0,9–1,1. Split validasi dan test tidak menerima augmentasi. Normalisasi ImageNet ditangani oleh transform internal Faster R-CNN.

Pelatihan menggunakan mixed precision pada CUDA, *gradient clipping* dengan norma maksimum 10, dan seleksi checkpoint berdasarkan mAP@[0.50:0.95] pada split validasi.

### Baseline dan fine-tuning

Checkpoint baseline terbaik diperoleh pada epoch 19 dengan mAP validasi sekitar 0,7088. Fine-tuning dimulai dari checkpoint tersebut, bukan dari bobot awal:

| Pengaturan fine-tuning | Nilai |
| --- | --- |
| Epoch maksimum | 10 epoch tambahan |
| Learning rate | 0,0001 |
| Scheduler | StepLR setiap 5 epoch, gamma 0,1 |
| Early stopping | 5 epoch tanpa peningkatan mAP validasi |
| Seleksi model | Hanya checkpoint yang melampaui mAP validasi terbaik sebelumnya |

Split test tetap tidak digunakan untuk pemilihan checkpoint. Bila fine-tuning tidak melampaui mAP validasi baseline, notebook mempertahankan checkpoint baseline sebagai model akhir.

## Hasil Faster R-CNN

Hasil berikut berasal dari evaluasi notebook pada split test yang terkunci.

| Model | mAP@[0.50:0.95] | mAP@0.50 | mAP@0.75 |
| --- | ---: | ---: | ---: |
| Baseline | 0,7157 | 0,9562 | 0,8791 |
| Fine-tuned (model akhir) | **0,7200** | **0,9565** | **0,8910** |

Model *fine-tuned* dipilih sebagai model akhir. Metrik YOLOv11 belum dicantumkan di README ini karena tidak tersedia dalam notebook Faster R-CNN yang menjadi acuan pembaruan ini.

### Artefak keluaran Faster R-CNN

Keluaran baseline disimpan di `/kaggle/working/faster_rcnn_sawit/`:

```text
best_faster_rcnn_resnet50_fpn.pth
last_faster_rcnn_resnet50_fpn.pth
history.json
learning_curve.png
final_metrics.json
```

Keluaran fine-tuning disimpan di `/kaggle/working/faster_rcnn_sawit_finetune/`:

```text
best_faster_rcnn_finetuned.pth
history_finetune.json
final_metrics_finetune.json
baseline_vs_finetune_corrected.png
error_analysis/
```

Direktori `error_analysis/` berisi `all_errors.csv`, `miss_by_luminance.csv`, serta panel contoh untuk tandan terlewat, salah kelas, dan box ganda.

### Analisis error Faster R-CNN

Audit error menggunakan IoU ≥ 0,50 dan ambang skor ≥ 0,50 pada split test. Error dikelompokkan sebagai:

- **tandan terlewat** — tidak ada prediksi kelas yang sama pada IoU ≥ 0,50;
- **salah kelas** — prediksi menutupi ground truth pada IoU ≥ 0,50, tetapi labelnya berbeda;
- **box ganda** — lebih dari satu prediksi dari kelas yang sama menutupi satu ground truth.

Hasil analisis menunjukkan box ganda jarang terjadi. Kesalahan yang lebih dominan adalah pertukaran kelas kematangan, terutama antara `mentah` dan `kurang masak`. Banyak prediksi sudah melokalisasi tandan dengan tepat, tetapi memberi kelas yang salah dengan skor tinggi. Kolom `occlusion_review` sengaja dibiarkan untuk pemeriksaan manual karena tingkat keterhalangan daun tidak dapat disimpulkan hanya dari bounding box.

Simpan hiperparameter, *seed*, versi library, dan informasi akselerator pada notebook atau hasil eksperimen masing-masing.

## Ringkasan EDA dan audit data

EDA dijalankan pada bounding box hasil standardisasi dan mencakup:

- ukuran split dan jumlah objek per gambar;
- distribusi kelas per split;
- luas serta rasio aspek bounding box;
- heatmap posisi pusat bounding box;
- brightness dan rata-rata kanal RGB pada sampel acak setiap split;
- pengecekan duplikat identik lintas split.

Hasil audit menunjukkan seluruh gambar yang lolos berukuran 640 × 640 dan tidak ada duplikat berkas yang identik persis lintas split (`exact_cross_split_duplicates.json` kosong). Distribusi brightness pada train, validasi, dan test saling bertumpang tindih dengan rata-rata yang berdekatan. Kanal merah rata-rata paling tinggi, disusul hijau lalu biru—konsisten dengan dominasi tone hangat/kuning-hijau pada citra sawit.

## Aturan perbandingan yang adil

1. Gunakan split, mapping kelas, dan bounding box hasil standardisasi yang sama untuk kedua model.
2. Laporkan mAP@0.5, mAP@0.5:0.95, precision, dan recall pada test set yang sama.
3. Simpan hiperparameter, versi library, *seed*, dan perangkat akselerator pada tiap eksperimen.
4. Augmentasi online boleh berbeda antar-framework, tetapi hanya diterapkan pada train dan harus didokumentasikan.
5. Jangan menggunakan test set untuk memilih *epoch*, *confidence threshold*, maupun hiperparameter.

## Keterbatasan

- *Stretch resize* dari Roboflow dapat mengubah rasio aspek objek, meskipun diterapkan konsisten pada seluruh split.
- Dataset berasal dari frame video. Pengecekan hash hanya mendeteksi duplikat yang identik persis, bukan seluruh frame yang sangat mirip.
- Konversi poligon ke bounding box menghilangkan detail kontur, namun konsisten dengan tujuan deteksi objek berbasis bounding box.
- Kualitas serta definisi anotasi mengikuti dataset publik dan perlu dicatat dalam pembahasan hasil.

## Lisensi

Repositori ini menggunakan [MIT License](LICENSE).
