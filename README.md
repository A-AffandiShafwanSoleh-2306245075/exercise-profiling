# Performance Testing & Profiling Report


## Deskripsi Project

Pada praktikum ini, dilakukan pengujian performa terhadap sebuah aplikasi berbasis **Spring Boot** yang terhubung dengan database **PostgreSQL**. Aplikasi ini menyediakan beberapa endpoint REST yang digunakan untuk mengambil data mahasiswa dengan berbagai tingkat kompleksitas query.

Tujuan utama dari pengujian ini adalah untuk memahami bagaimana sistem bereaksi ketika menerima banyak request secara bersamaan (concurrent users), serta mengidentifikasi bagian mana dari sistem yang menjadi bottleneck. Selain itu, hasil dari pengujian ini juga akan digunakan sebagai dasar untuk tahap selanjutnya, yaitu **profiling dan optimasi performa**.

---

## Endpoint yang Diuji

Dalam pengujian ini, terdapat tiga endpoint utama yang diuji, masing-masing memiliki karakteristik dan beban kerja yang berbeda.

| Endpoint | Deskripsi | Beban |
|----------|-----------|-------|
| `/all-student` | Mengambil seluruh data mahasiswa termasuk relasi antar tabel | Berat |
| `/all-student-name` | Hanya mengambil nama mahasiswa tanpa data tambahan | Ringan |
| `/highest-gpa` | Mencari mahasiswa dengan nilai GPA tertinggi (agregasi sederhana) | Ringan |

---

## ⚙Konfigurasi Pengujian

Pengujian dilakukan menggunakan **Apache JMeter** dengan konfigurasi sebagai berikut:

| Parameter | Nilai |
|-----------|-------|
| Jumlah User (Threads) | 10 |
| Ramp-up Period | 1 detik |
| Loop Count | 1 |

Konfigurasi ini mensimulasikan kondisi di mana 10 pengguna mengakses sistem hampir secara bersamaan.

---

## Hasil Pengujian Menggunakan JMeter (GUI)

### Summary Report

![Summary Report](assets/readme1.1.png)

Summary Report memberikan gambaran statistik keseluruhan dari hasil pengujian. Dari data yang diperoleh:
- Endpoint `/all-student` memiliki rata-rata waktu respons **paling tinggi** dibandingkan endpoint lainnya, mengindikasikan bottleneck utama sistem.
- Terdapat **error rate sekitar 10%**, menunjukkan ketidakstabilan sistem ketika menerima beban.
- **Throughput** yang dihasilkan relatif rendah, berarti jumlah request yang dapat diproses per satuan waktu masih terbatas.

---

### View Results Tree

![View Results Tree](assets/readme1.2.png)

Pada tampilan View Results Tree, dapat dilihat bahwa:
- Sebagian besar request berhasil dieksekusi dengan status **200 (OK)** ✅ (ditandai warna hijau).
- Terdapat beberapa request awal yang **gagal** ❌ (ditandai warna merah).

Kegagalan di awal ini kemungkinan besar disebabkan oleh kondisi awal server yang belum stabil, seperti proses inisialisasi koneksi database atau *cold start* aplikasi. Setelah beberapa saat, sistem mulai stabil dan mampu menangani request dengan baik.

---

### View Results in Table

![View Results Table](assets/readme1.3.png)

Dari tampilan tabel hasil pengujian, terlihat perbedaan waktu respons yang sangat signifikan antar endpoint:

- **`/all-student`** — Waktu respons sangat tinggi, bahkan mencapai ratusan ribu milidetik. Ini menunjukkan query yang tidak efisien atau terlalu banyak data yang diambil sekaligus.
- **`/all-student-name`** — Waktu respons jauh lebih rendah, berada di kisaran beberapa ribu milidetik sesuai ekspektasi.
- **`/highest-gpa`** — Performa terbaik dengan waktu respons paling rendah, hanya ratusan milidetik. Query agregasi sederhana jauh lebih efisien.

---

### Graph Results

![Graph Results](assets/readme1.4.png)

Grafik hasil pengujian menunjukkan:
- Adanya **fluktuasi yang cukup besar** pada waktu respons, terutama pada endpoint `/all-student`.
- Variasi yang tinggi ini mengindikasikan **performa sistem tidak stabil** ketika menerima beban meningkat.
- Bottleneck pada database kemungkinan disebabkan oleh query lambat, *locking*, atau penggunaan resource yang tidak efisien.

---

## Pengujian Menggunakan JMeter CLI

Selain menggunakan GUI, pengujian juga dilakukan menggunakan mode CLI dengan perintah:

```bash
jmeter -n -t "Test Plan.jmx" -l test_result_log.jtl
```

### Hasil CLI

![CLI Output](assets/readme1.5.png)

Pada mode CLI, hasil yang diperoleh sangat berbeda dibandingkan GUI. **Semua request menghasilkan status 500 (Internal Server Error)** dengan waktu respons sekitar 30 detik.

---

### File Log (.jtl)

![JTL Result](assets/readme1.6.png)

Dari file log, terlihat bahwa **seluruh endpoint gagal diproses**. Perbedaan hasil antara GUI dan CLI terjadi karena:

| Mode | Perilaku |
|------|----------|
| **GUI** | Memberikan beban secara bertahap (lebih "ramah") |
| **CLI** | Langsung memberikan beban penuh tanpa jeda → sistem overload |

---

## Analisis Masalah

Berdasarkan seluruh hasil pengujian, terdapat beberapa masalah utama dalam sistem:

1. **Bottleneck pada `/all-student`** — Endpoint ini mengambil data dalam jumlah besar dengan query yang tidak efisien, menyebabkan waktu respons sangat tinggi.
2. **Sistem tidak siap menangani beban langsung** — Hasil CLI dengan error 100% menunjukkan aplikasi belum siap untuk kondisi *real-world* dengan banyak concurrent user.
3. **Ketidakstabilan sistem** — Adanya error rate menunjukkan sistem masih memiliki kelemahan dalam menangani request secara konsisten.

---

## Kesimpulan

Secara keseluruhan, performa sistem masih **belum optimal**:

- Endpoint dengan beban besar menunjukkan waktu respons yang sangat tinggi dan menjadi bottleneck utama.
- Sistem tidak stabil ketika menerima beban secara langsung, dibuktikan oleh hasil pengujian CLI dengan **error rate 100%**.

---