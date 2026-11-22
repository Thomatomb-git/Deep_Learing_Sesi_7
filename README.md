# **Deep Learning Sesi 07**

**Nama:** Frederick Kamsono  
**NIM:** 2802549205  
**Kelas:** LA01  

---

## Deskripsi Singkat
Repositori ini berisi implementasi dan evaluasi komparatif enam arsitektur model generatif (**Vanilla Autoencoder**, **Sparse Autoencoder**, **Variational Autoencoder / VAE**, **Vanilla GAN**, **DCGAN**, dan **DDPM**) beserta eksperimen pembuktian batasan teoretis dan empiris **MLP Classifier** untuk *image generation* pada dataset **Fashion-MNIST** ($28 \times 28$ grayscale, split 80/10/10, `seed = 42`)[cite: 1, 2, 3].

## Struktur Repositori
```text
├── init.ipynb                                           # Notebook utama (data pipeline, training, evaluasi, visualisasi)
├── outputs/
│   ├── figures/                                         # 5 figur visualisasi komparatif (300 DPI)
│   └── logs/                                            # Log riwayat loss (.json) & tabel metrik (.csv)
├── laporan.tex                                          # Sumber dokumen LaTeX laporan ilmiah
├── laporan.pdf     # Laporan akhir PDF
├── requirements.txt                                     # Daftar dependensi pustaka Python
└── README.md
```

## Ringkasan Hasil Utama (Test Set)
* **MLP Classifier Accuracy:** `88.61%` (Frozen Latent Decoder MSE: `0.01758`, PSNR: `17.55 dB`, SSIM: `0.6556`)
* **Rekonstruksi Terbaik:** Vanilla AE (`MSE: 0.00838`, `PSNR: 20.77 dB`, `SSIM: 0.7895`) & Sparse AE (`MSE: 0.00844`, `PSNR: 20.74 dB`, `SSIM: 0.7875`)
* **Kualitas Generatif Terbaik:** DCGAN (`FID: 29.10`, `Latency: 1.36 ms / batch 64`)
* **Stabilitas Training Tertinggi:** DDPM (`FID: 91.54`, `Latency: 727.22 ms / batch 64`)

## Rekap Hasil Evaluasi Test Set (Fashion-MNIST, Seed 42)

### A. Pembuktian Batasan MLP Classifier
- Akurasi Test Set MLP Classifier (784 -> 256 -> 64 -> 10): 88.61%
- Rekonstruksi Frozen Latent Decoder (z_MLP): MSE = 0.01758 | PSNR = 17.55 dB | SSIM = 0.6556
- Perbandingan vs Vanilla AE (z_AE): MSE melonjak +109.78% (2.10x lebih buruk), PSNR turun -3.22 dB, SSIM turun -0.1339 (-16.96%).
- Perbandingan vs VAE (z_VAE): MSE 1.13x (+12.98%) lebih buruk meski VAE dibebani regularisasi KL Divergence.

### B. Tabel Komparasi 6 Model Generatif
1. Vanilla Autoencoder (Latent 64):
   - Train Time: 6.71 s/epoch | Latency (batch 64): 0.21 ms
   - Test MSE: 0.00838 | Test PSNR: 20.77 dB | Test SSIM: 0.7895 | FID: N/A (Latent Voids)
   - Loss: turun mulus 0.03488 -> 0.00813 (25 epoch)

2. Sparse Autoencoder (Latent 64, L1 lambda=1e-4):
   - Train Time: 6.86 s/epoch | Latency (batch 64): 0.19 ms
   - Test MSE: 0.00844 (+0.72% vs Vanilla AE) | Test PSNR: 20.74 dB | Test SSIM: 0.7875 | FID: N/A (Latent Voids)
   - Loss: turun mulus 0.03495 -> 0.00831 (25 epoch)

3. Variational Autoencoder / VAE (Latent 64):
   - Train Time: 7.59 s/epoch | Latency (batch 64): 0.10 ms (tercepat)
   - Test MSE: 0.01556 (1.86x vs Vanilla AE) | Test PSNR: 18.08 dB | Test SSIM: 0.6801 | FID: 94.12
   - Loss (ELBO): turun mulus 299.61 -> 241.00 (25 epoch)

4. Vanilla GAN / MLP (Prior Noise 100):
   - Train Time: 9.14 s/epoch | Latency (batch 64): 0.13 ms | FID: 147.31
   - Dinamika Loss: G melonjak ke 2.427 di epoch 2 saat D turun ke 0.616; akhir epoch 30 G = 1.093, D = 1.163

5. DCGAN / Convolutional (Prior Noise 100):
   - Train Time: 9.21 s/epoch | Latency (batch 64): 1.36 ms (~0.021 ms/citra) | FID: 29.10 (80.25% / 5.06x lebih baik dari Vanilla GAN)
   - Dinamika Loss: jauh lebih stabil; G 1.685 -> 1.015, D 0.670 -> 1.192 (30 epoch)

6. DDPM / Mini U-Net (28x28, T=200):
   - Train Time: 15.17 s/epoch | Latency (batch 64): 727.22 ms (~11.36 ms/citra; 534.7x lebih lambat dari DCGAN & 7272.2x lebih lambat dari VAE) | FID: 91.54
   - Loss (Noise MSE): paling stabil, turun monoton 0.15752 -> 0.07988 (min 0.07898 di epoch 24)
