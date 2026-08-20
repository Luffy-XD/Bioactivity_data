<div align="center">

# 🧬 PROGRAM MODEL MLP

### Prediksi Bioaktivitas Senyawa AChE menggunakan Multi-Layer Perceptron

![Python](https://img.shields.io/badge/Python-3.10-blue?logo=python&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.7-f9c010?logo=scikit-learn&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-3.0-000000?logo=flask&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)
![RDKit](https://img.shields.io/badge/RDKit-Molecular_Representation-2cafeb?logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZD0iTTEyIDJMMyAxNmg0djZoMTB2LTZoNGwxLTExem0tNCAxOGgtMnYtMmg0djJoLTJ2LTJoLTJ2MmgweiIgZmlsbD0id2hpdGUiLz48L3N2Zz4=&logoColor=white)

Sistem prediksi bioaktivitas senyawa terhadap target **Acetylcholinesterase (AChE)** menggunakan model **Multi-Layer Perceptron (MLP)**.

Proyek ini mencakup alur kerja mulai dari pengumpulan data dari database [ChEMBL](https://www.ebi.ac.uk/chembl/),
pra-pemrosesan fitur molekul ([Mordred](https://github.com/mordred-descriptor/mordred)),
pelatihan model MLP (baseline + 5 varian tuning),
hingga demonstrasi interaktif via CLI dan dashboard web.

---

</div>

## 📂 Struktur Proyek

```
PROGRAM_MODEL_MLP/
│
├── 📁 pengumpulan_dan_prapemrosesan_data/
│   ├── pengumpulan_data_dan_prapemrosesan.ipynb   ← Pengumpulan data & pra-pemrosesan
│   ├── bioactivity_data.csv                       ← Data mentah dari ChEMBL
│   └── bioactivity_ready_for_model.csv            ← Dataset siap model
│
└── 📁 pelatihan_model/
    ├── model_mlp_default.ipynb                    ← Pelatihan MLP baseline
    ├── model_mlp_tuning_0.ipynb                   ← Tuning 0
    ├── model_mlp_tuning_1.ipynb                   ← Tuning 1
    ├── model_mlp_tuning_2.ipynb                   ← Tuning 2
    ├── model_mlp_tuning_3.ipynb                   ← Tuning 3
    ├── model_mlp_tuning_4.ipynb                   ← Tuning 4
    │
    └── 📁 demo_model/
        ├── demo_model_mlp_default.pkl             ← Model tersimpan (default)
        ├── demo_model_mlp_tuning_0.pkl            ← Model terbaik (90.76%)
        ├── demo_model_mlp_tuning_1.pkl
        ├── demo_model_mlp_tuning_2.pkl
        ├── demo_model_mlp_tuning_3.pkl
        ├── demo_model_mlp_tuning_4.pkl
        │
        ├── run_model_mlp_default.ipynb            ← Demo notebook (default)
        ├── run_model_mlp_tuning_0.ipynb           ← Demo notebook (tuning 0)
        ├── run_model_mlp_tuning_1.ipynb           ← Demo notebook (tuning 1)
        ├── run_model_mlp_tuning_2.ipynb           ← Demo notebook (tuning 2)
        ├── run_model_mlp_tuning_3.ipynb           ← Demo notebook (tuning 3)
        ├── run_model_mlp_tuning_4.ipynb           ← Demo notebook (tuning 4)
        │
        └── 📁 main_demo/
            ├── main.py                            ← Aplikasi CLI (terminal interaktif)
            ├── app.py                             ← Backend REST API (Flask)
            ├── main.html                          ← Dashboard web (SPA dark theme)
            └── requirements.txt                   ← Dependensi Flask app
```

---

## 🔄 Alur Kerja

### 1. Pengumpulan Data

Data dikumpulkan dari database [ChEMBL](https://www.ebi.ac.uk/chembl/) menggunakan `chembl_webresource_client`.

| Parameter | Nilai |
|:---|:---|
| **Target** | Acetylcholinesterase ([CHEMBL220](https://www.ebi.ac.uk/chembl/target_report_card/CHEMBL220/)) |
| **Organisme** | *Homo sapiens* |
| **Tipe data** | IC50 (`standard_type="IC50"`) |

---

### 2. Pra-pemrosesan Data

| Tahap | Keterangan |
|:---|:---|
| Seleksi kolom | `molecule_chembl_id`, `canonical_smiles`, `standard_value` |
| Pelabelan kelas | **active** (IC50 ≤ 1.000 nM) · **inactive** (IC50 ≥ 10.000 nM) · intermediate dihapus |
| Konversi SMILES | Menggunakan RDKit (`Chem.MolFromSmiles`) |
| Komputasi deskriptor | **Mordred** — 1.800+ deskriptor molekul (2D/3D) |
| Pembersihan | Hapus nilai `inf` / `NaN`, imputasi median |
| Standardisasi | `StandardScaler` (zero-mean, unit-variance) |

> **Output**: `bioactivity_ready_for_model.csv` — **4.113 sampel** × **1.523 deskriptor** fitur

---

### 3. Pelatihan Model

Semua model menggunakan **`MLPClassifier`** (scikit-learn) dengan pipeline `StandardScaler → MLPClassifier`.
Split data: **80 / 20** (`random_state=42`, stratified).

#### MLP Default (Baseline)

| Parameter | Nilai |
|:---|:---|
| `hidden_layer_sizes` | `(100,)` |
| `activation` | `relu` |
| `solver` | `adam` |
| **Akurasi** | **90,61 %** |

#### MLP Tuning (Hyperparameter Optimization)

Optimasi menggunakan **`RandomizedSearchCV`**

| Pengaturan | Nilai |
|:---|:---|
| Jumlah iterasi | 100 |
| Cross-validation | 10-fold `StratifiedKFold` |
| Scoring | Akurasi |
| Total fits per varian | 1.000 |

**Ruang pencarian:**

```python
param_distributions = {
    'hidden_layer_sizes': [(50,), (100,), (150,), (100, 50), (150, 100, 50)],
    'activation':         ['relu', 'tanh', 'logistic'],
    'solver':             ['adam', 'sgd'],
    'alpha':              [0.0001, 0.001, 0.01],
    'learning_rate_init': [0.001, 0.01],
    'batch_size':         [32, 64, 128],
    'max_iter':           [100]
}
```

#### Hasil Perbandingan

| Varian | `random_state` | `hidden_layer_sizes` | `activation` | `solver` | `alpha` | `lr_init` | `batch_size` | **Akurasi** |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **🏆 Tuning 0** | **0** | **(150,)** | **logistic** | **adam** | **0.001** | **0.001** | **32** | **90,76 %** |
| Tuning 1 | 1 | (150, 100, 50) | tanh | sgd | 0.001 | 0.001 | 32 | 89,60 % |
| Tuning 2 | 2 | (100, 50) | tanh | adam | 0.01 | 0.001 | 128 | 88,82 % |
| Tuning 3 | 3 | (150, 100, 50) | relu | sgd | 0.01 | 0.01 | 64 | 88,66 % |
| Tuning 4 | 4 | (150, 100, 50) | relu | sgd | 0.01 | 0.01 | 64 | 88,74 % |

> **Model terbaik: Tuning 0** — Akurasi **90,76 %** dengan arsitektur hidden layer tunggal (150 neuron), aktivasi logistik, dan solver Adam.

---

### 4. Evaluasi Model

| Metrik | Deskripsi |
|:---|:---|
| **Classification Report** | Akurasi, presisi, recall, F1-score per kelas |
| **Confusion Matrix** | Heatmap prediksi vs label aktual |
| **ROC-AUC** | Kurva Receiver Operating Characteristic dan nilai AUC |
| **SHAP** | Analisis kontribusi global setiap deskriptor terhadap prediksi |

---

### 5. Penyimpanan Model

Model disimpan menggunakan `joblib` sebagai file `.pkl` berisi bundle dictionary:

```python
{
    "model":          mlp_classifier,
    "scaler":         standard_scaler,
    "label_encoder":  label_encoder,
    "features":       fitur_list,
    "X_train":        X_train,
    "X_test":         X_test,
    "y_train":        y_train,
    "y_test":         y_test,
    "model_name":     "MLP Tuning 0"
}
```

---

## 🚀 Aplikasi Demo

### A. CLI — Terminal Interaktif

> **File**: `pelatihan_model/demo_model/main_demo/main.py`

Aplikasi terminal interaktif menggunakan library **Rich** untuk antarmuka visual di terminal.

| Fitur | Deskripsi |
|:---|:---|
| Banner ASCII art | Arsitektur MLP yang dianimasi warna-warni |
| Menu utama | Prediksi senyawa · Statistik dataset · Database hasil prediksi |
| Prediksi batch | Seluruh dataset → `hasil_prediksi_aktif.csv` & `hasil_prediksi_inaktif.csv` |
| Prediksi baris | Analisis satu baris data spesifik |
| Grafik distribusi | Animated bar (aktif vs inaktif, probabilitas prediksi) |
| MLP diagram | Beranimasi sesuai tahapan inferensi (input → hidden → output) |

**Jalankan:**

```bash
cd pelatihan_model/demo_model/main_demo
pip install rich
python main.py
```

---

### B. Dashboard Web — Flask + HTML SPA

> **Backend**: `pelatihan_model/demo_model/main_demo/app.py`
> **Frontend**: `pelatihan_model/demo_model/main_demo/main.html`

Dashboard web SPA (Single Page Application) dengan **dark theme**. Backend Flask menyediakan REST API untuk prediksi, dan frontend menampilkan visualisasi real-time.

#### Halaman Dashboard

| Halaman | Deskripsi |
|:---|:---|
| **Dashboard** | Statistik ringkasan, histogram probabilitas, donut chart rasio kelas, arsitektur MLP, confidence gauge, prediksi terbaru |
| **Prediksi Senyawa** | Mode batch (seluruh dataset) & mode baris spesifik, progress bar animasi, detail probabilitas |
| **Dataset** | Ringkasan statistik, grafik distribusi aktif vs inaktif, pratinjau deskriptor |
| **Hasil Prediksi** | Tabel dari `hasil_prediksi_aktif.csv` / `hasil_prediksi_inaktif.csv`, tombol hapus permanen |
| **Model & Config** | Status pemuatan model, detail info bundle `.pkl` |

#### REST API Endpoints

| Endpoint | Method | Deskripsi |
|:---|:---:|:---|
| `/` | `GET` | Serve halaman `main.html` |
| `/api/status` | `GET` | Status model, data, jumlah hasil prediksi |
| `/api/dashboard` | `GET` | Data ringkasan dashboard |
| `/api/predict-batch` | `POST` | Jalankan prediksi batch seluruh dataset |
| `/api/predict-row` | `POST` | Prediksi baris spesifik (`row_index`) |
| `/api/results/<tipe>` | `GET` | Ambil hasil prediksi (`aktif` / `inaktif`) |
| `/api/results/delete` | `POST` | Hapus semua file hasil prediksi |
| `/api/dataset/stats` | `GET` | Statistik dataset |

**Jalankan:**

```bash
cd pelatihan_model/demo_model/main_demo

# Aktifkan virtual environment (opsional)
source .venv/bin/activate      # Linux / macOS
# .venv\Scripts\activate       # Windows

# Install dependensi
pip install -r requirements.txt

# Jalankan server
python app.py
```

> 🌐 Dashboard dapat diakses di **http://localhost:5000**

---

## ⚙️ Persyaratan Sistem

### Notebook (Pelatihan & Pra-pemrosesan)

| Library | Kegunaan |
|:---|:---|
| `chembl_webresource_client` | Pengumpulan data dari ChEMBL |
| `rdkit-pypi` | Konversi SMILES ke objek molekul |
| `mordred` | Komputasi deskriptor molekul (1.800+) |
| `scikit-learn` | MLPClassifier, StandardScaler, evaluasi model, RandomizedSearchCV |
| `shap` | Analisis kontribusi fitur (SHAP values) |
| `pandas` | Manipulasi data |
| `numpy` | Komputasi numerik |
| `matplotlib` | Visualisasi (confusion matrix, ROC curve, SHAP plot) |
| `seaborn` | Statistical visualization |
| `ipywidgets` | Interaktivitas di Jupyter Notebook |
| `joblib` | Penyimpanan & pemuatan model (`.pkl`) |

### Aplikasi Demo — Flask Web

```
flask>=3.0.0
pandas>=2.0.0
joblib>=1.3.0
numpy>=1.24.0
scikit-learn>=1.3.0
```

### Aplikasi Demo — CLI

```
rich
```

---

## 👤 Peneliti

| Peran | Nama |
|:---|:---|
| Peneliti QSAR | **Mun Awani** |

---

## 📜 Lisensi

Proyek ini dikembangkan untuk keperluan **penelitian / tugas akhir**.
Hubungi peneliti untuk informasi lebih lanjut mengenai penggunaan dan distribusi.
