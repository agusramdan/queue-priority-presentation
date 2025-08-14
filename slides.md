---
marp: true
theme: default
class: invert
backgroundColor: black
color: #00FF00
style: |
  section {
    font-family: 'Courier New', Courier, monospace;
    background-color: #000000;
    color: #00FF00;
  }
---

# Background Process di Odoo


"Background jobs: unseen, unheard, unstoppable.

---

## Pendahuluan
Background process = proses yang berjalan di belakang layar.

**Tujuan:**
- Mengurangi beban proses di UI.
- Menjalankan tugas rutin secara otomatis.
- Menangani proses berat secara asinkron.

---

## Cron (Scheduled Actions)
Menjalankan fungsi Python terjadwal menggunakan `ir.cron`.

**Alur:**
1. ir.cron disimpan di DB.
2. Worker cron memeriksa interval.
3. Eksekusi fungsi Python.

**Kelebihan:** bawaan Odoo, mudah konfigurasi.  
**Kekurangan:** blocking jika proses berat.

---

## Queue Job
Menjalankan proses secara asinkron lewat antrian job.

**Alur:**
1. Job dimasukkan ke antrian.
2. Worker mengambil job.
3. Eksekusi di background.

**Kelebihan:** tidak blocking, cocok proses berat.  
**Kekurangan:** perlu modul tambahan.

---

## Background Process dari UI
Proses berat yang dipicu oleh aksi user di UI.

Tidak dijalankan langsung di thread request/response.

**Alur:**
1. User klik tombol / submit form.
2. Server membuat job di queue.
3. Worker memproses job.
4. User mendapat notifikasi / hasil.

**Manfaat:** UI tetap responsif.

---

## Background Process dari UI (2)

**Diagram ASCII:**

[User Click]
|
v
[Odoo Controller]
|
v
[Create Job in Queue]
|
v
[Background Worker]
|
v
[Process Job]
|
v
[Notify User / Update Record]

---

## Queue dengan Prioritas
Queue memproses job berdasarkan prioritas, bukan urutan masuk.

**Angka lebih kecil = prioritas lebih tinggi.**

Contoh urutan eksekusi: **B(0) → C(5) → A(10)**.

**Kapan digunakan:**
- Proses kritis (notifikasi real-time).
- Sinkronisasi data penting.
- Job dengan SLA tinggi.


---

## Queue dengan Waktu Eksekusi (ETA)
- ETA = Estimated Time of Arrival (jadwal eksekusi).
- Gunakan ketika:
  - Job hanya boleh berjalan setelah waktu tertentu.
  - Contoh: kirim reminder 2 jam sebelum meeting.
- Implementasi di `queue_job`:
  ```python
  self.with_delay(
      priority=10,
      eta=datetime.now() + timedelta(hours=2)
  ).my_function()
Worker hanya akan memproses job jika eta <= waktu sekarang.


---

## Cron Dispatch Job
Menggabungkan cron sebagai pemicu dan queue sebagai eksekutor.

**Alur:**
1. Cron dijalankan sesuai jadwal.
2. Cron membuat job di queue.
3. Worker queue memproses job.

**Kelebihan:** ringan di cron, scalable untuk job berat.

---

## Perbandingan Metode

| Metode           | Terjadwal | Async | Bawaan | Skala Besar |
|------------------|-----------|-------|--------|-------------|
| Cron             | ✅        | ❌    | ✅     | ⚠           |
| Queue Job        | ❌        | ✅    | ❌     | ✅           |
| Background UI    | ❌        | ✅    | ❌     | ✅           |
| Priority Queue   | ❌        | ✅    | ❌     | ✅           |
| Cron Dispatch    | ✅        | ✅    | ❌     | ✅           |

---

## Best Practice
- Gunakan Cron untuk tugas ringan & rutin.
- Gunakan Queue Job untuk tugas berat tanpa jadwal.
- Gunakan Background UI untuk proses berat yang dipicu user.
- Gunakan Priority Queue jika ada job penting yang harus dikerjakan dulu.
- Gunakan Cron Dispatch Job untuk proses berat yang butuh jadwal.


---

## END Q/A

Like daemons in the system, silent workers keep the world running.


