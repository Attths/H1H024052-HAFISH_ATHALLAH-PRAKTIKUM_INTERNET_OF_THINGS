<h1>Praktikum Percobaan </h1>
<h2>Tujuan Praktikum</h2>
Percobaan 5B bertujuan untuk memahami dan mengimplementasikan proses akuisisi, analisis, dan visualisasi data IoT pada sisi komputer menggunakan python. Pada percobaan ini, data suhu diterima melalui protokol MQTT, kemudian disimpan ke dalam file CSV untuk dianalisis menggunakan statistik sederhana dan metode z-score guna mendeteksi anomali Hasil analisis selanjutnya divisualisasikan dalam bentuk grafik untuk mengamati perubahan suhu dan data yang terindikasi sebagai anomali.
 
<hr>
<h2>Penjelasan Code</h2>

```cpp
import json
import time
import paho.mqtt.client as mqtt
import pandas as pd
import matplotlib.pyplot as plt

BROKER = "broker.hivemq.com"
PORT = 1883
TOPIC = "unsoed/tk245004/kelompok1/data"  # Sesuaikan dengan topicData ESP32
JUMLAH_SAMPEL = 20                        # Jumlah data yang dikumpulkan

data = []
```
Bagian ini mengimpor library yang dibutuhkan: json untuk mengolah payload, time untuk mencatat waktu penerimaan, ```paho.mqtt.client``` untuk komunikasi MQTT, pandas untuk analisis data, dan matplotlib untuk grafik. Konfigurasi broker (HiveMQ, port 1883), topic yang di-subscribe, dan jumlah sampel yang dikumpulkan (20) didefinisikan sebagai konstanta. List kosong data menampung seluruh pembacaan yang diterima.

```cpp
def on_connect(client, userdata, flags, reason_code, properties):
    print("Terhubung ke broker, kode:", reason_code)
    client.subscribe(TOPIC)
```
Fungsi ```on_connect()``` dipanggil otomatis begitu koneksi ke broker berhasil. Fungsi ini mencetak kode status koneksi lalu melakukan subscribe ke TOPIC. Subscribe ditaruh di sini, bukan di luar fungsi, agar langganan dibuat ulang setiap kali koneksi tersambung kembali.

```cpp
def on_message(client, userdata, msg):
    try:
        payload = json.loads(msg.payload.decode())
        suhu = float(payload["suhu"])
    except (ValueError, KeyError):
        print("Data tidak valid, diabaikan")
        return

    data.append({
        "waktu": time.strftime("%Y-%m-%d %H:%M:%S"), 
        "suhu": suhu
    })
    print(f"[{len(data)}/{JUMLAH_SAMPEL}] suhu = {suhu}")

    if len(data) >= JUMLAH_SAMPEL:
        client.disconnect()
```
Fungsi ```on_message()``` dipanggil setiap ada pesan masuk. Payload di-decode menjadi teks, di-parse sebagai JSON, lalu nilai "suhu" diambil dan diubah menjadi float. Blok try-except menangkap pesan yang bukan JSON valid (ValueError) atau tidak memiliki key "suhu" (KeyError), sehingga data seperti itu diabaikan tanpa menghentikan program. Data valid disimpan ke list data bersama stempel waktu penerimaan. Ketika jumlah data sudah mencapai ```JUMLAH_SAMPEL```, ```client.disconnect()``` dipanggil untuk menghentikan akuisisi.

```cpp
client = mqtt.Client(mqtt.CallbackAPIVersion.VERSION2)
client.on_connect = on_connect
client.on_message = on_message
client.connect(BROKER, PORT)
client.loop_forever()
```
Tahap 1 membuat objek client MQTT (dengan callback API versi 2), mendaftarkan kedua fungsi callback, lalu terhubung ke broker. loop_forever() menjaga program tetap menunggu pesan masuk. Perintah ini bersifat blocking dan baru selesai setelah client.disconnect() dipanggil, sehingga kode setelahnya (analisis dan visualisasi) baru berjalan setelah 20 data terkumpul.

```cpp
df = pd.DataFrame(data)

rata2 = df["suhu"].mean()
std = df["suhu"].std()

if std == 0:
    df["zscore"] = 0.0
else:
    df["zscore"] = ((df["suhu"] - rata2) / std).round(4)

df["anomali"] = df["zscore"].abs() > 2
df["status_anomali"] = df["anomali"].apply(lambda x: "Anomali" if x else "Normal")
```
Tahap 2 mengubah list data menjadi DataFrame, lalu menghitung rata-rata dan simpangan baku suhu. Z-score dihitung dengan rumus (suhu − rata-rata) / simpangan baku dan dibulatkan 4 angka desimal. Pengecekan std == 0 mencegah pembagian dengan nol ketika semua data bernilai sama. Data dengan nilai absolut z-score di atas 2 ditandai True pada kolom anomali, lalu kolom status_anomali mengubahnya menjadi label teks “Anomali” atau “Normal”.

```cpp
try:
    df["status_penyimpanan_csv"] = "Tersimpan"
    kolom_tabel = ["waktu", "suhu", "zscore", "status_anomali", "status_penyimpanan_csv"]
    df.to_csv("data_suhu.csv", columns=kolom_tabel, index=False)
    simpan_sukses = True
except Exception as e:
    df["status_penyimpanan_csv"] = f"Gagal ({e})"
    simpan_sukses = False

if simpan_sukses:
    print("\n✓ Status: Seluruh data pengamatan (20 data) berhasil disimpan ke 'data_suhu.csv'")
else:
    print("\n✗ Status: Terjadi kesalahan saat menyimpan ke CSV")
```
Tahap 3 menyimpan hasil analisis ke data_suhu.csv. Kolom yang disimpan dipilih lewat kolom_tabel, dan index=False menghilangkan nomor indeks dari file. Penyimpanan dibungkus try-except agar kegagalan (misalnya file sedang terbuka di Excel) tidak membuat program crash, dan statusnya dicatat pada variabel simpan_sukses lalu ditampilkan ke terminal.

```cpp
print(f"\n--- Ringkasan Statistik ---")
print(f"Rata-rata     : {rata2:.2f} °C")
print(f"Std deviasi   : {std:.2f}")
print(f"Minimum       : {df['suhu'].min():.2f} °C")
print(f"Maksimum      : {df['suhu'].max():.2f} °C")
print(f"Total Sampel  : {len(df)}")
print(f"Jumlah Anomali: {df['anomali'].sum()}\n")
```
Bagian ini mencetak ringkasan statistik ke terminal: rata-rata, simpangan baku, nilai minimum dan maksimum, jumlah sampel, serta jumlah anomali. Jumlah anomali didapat dari df['anomali'].sum(), karena nilai True dihitung sebagai 1.

```cpp
plt.figure(figsize=(10, 4))
plt.plot(df.index, df["suhu"], marker="o", label="Suhu")

anom = df[df["anomali"]]
plt.scatter(anom.index, anom["suhu"], color="red", s=100, label="Anomali", zorder=3)

plt.axhline(rata2, color="gray", linestyle="--", label="Rata-rata")
plt.xlabel("Urutan data")
plt.ylabel("Suhu (°C)")
plt.title("Data Suhu dan Deteksi Anomali")
plt.legend()
plt.tight_layout()
plt.savefig("grafik_suhu.png")
plt.show()
```
Tahap 4 membuat grafik berukuran 10×4 inci. Garis suhu digambar terhadap urutan data dengan penanda titik. Data yang terdeteksi anomali disaring ke anom lalu digambar sebagai titik merah besar, dan zorder=3 memastikan titik itu tampil di atas garis. Garis putus-putus abu-abu menunjukkan rata-rata suhu. Setelah label sumbu, judul, dan legenda diatur, grafik disimpan sebagai grafik_suhu.png dan ditampilkan di layar.

<hr>

<h2>Pertanyaan Praktikum Modifikasi Program</h2>
Modifikasi program agar menggunakan metode moving average (jendela 5 data) pada
data hasil CSV sebelum dihitung z-score-nya, kemudian bandingkan jumlah anomali
yang terdeteksi, dan berikan penjelasan di setiap baris kode yang ditambahkan dalam
bentuk README.md.

```cpp
import json
import time
import paho.mqtt.client as mqtt
import pandas as pd
import matplotlib.pyplot as plt

BROKER = "broker.hivemq.com"
PORT = 1883
TOPIC = "unsoed/tk245004/kelompok1/data"  # Sesuaikan dengan topicData ESP32
JUMLAH_SAMPEL = 20  # Jumlah data yang dikumpulkan

data = []

def on_connect(client, userdata, flags, reason_code, properties):
    print("Terhubung ke broker, kode:", reason_code)
    client.subscribe(TOPIC)

def on_message(client, userdata, msg):
    try:
        payload = json.loads(msg.payload.decode())
        suhu = float(payload["suhu"])
    except (ValueError, KeyError):
        print("Data tidak valid, diabaikan")
        return

    data.append({
        "waktu": time.strftime("%Y-%m-%d %H:%M:%S"),
        "suhu": suhu
    })

    print(f"[{len(data)}/{JUMLAH_SAMPEL}] suhu = {suhu}")

    if len(data) >= JUMLAH_SAMPEL:
        client.disconnect()

# --- Tahap 1: Akuisisi data dari MQTT ---
client = mqtt.Client(mqtt.CallbackAPIVersion.VERSION2)
client.on_connect = on_connect
client.on_message = on_message
client.connect(BROKER, PORT)
client.loop_forever()

# --- Tahap 2: Pengolahan DataFrame & Analisis Z-Score ---
df = pd.DataFrame(data)

rata2 = df["suhu"].mean()
std = df["suhu"].std()

# Kondisional perhitungan Z-Score
if std == 0:
    df["zscore"] = 0.0
else:
    df["zscore"] = ((df["suhu"] - rata2) / std).round(4)

# Kondisional Status Anomali (ambang batas z-score > 2 atau < -2)
df["anomali"] = df["zscore"].abs() > 2
df["status_anomali"] = df["anomali"].apply(lambda x: "Anomali" if x else "Normal")

# [MODIFIKASI] Menghitung moving average dengan jendela 5 data
# min_periods=1 agar data awal (kurang dari 5) tetap dihitung
df["suhu_ma"] = df["suhu"].rolling(
    window=5, min_periods=1
).mean()

# [MODIFIKASI] Deteksi anomali pada data suhu asli
rata2_asli = df["suhu"].mean()
std_asli = df["suhu"].std()
df["z_asli"] = (df["suhu"] - rata2_asli) / std_asli
df["anomali_asli"] = df["z_asli"].abs() > 2

# [MODIFIKASI] Deteksi anomali pada data hasil moving average
rata2_ma = df["suhu_ma"].mean()
std_ma = df["suhu_ma"].std()
df["z_ma"] = (df["suhu_ma"] - rata2_ma) / std_ma
df["anomali_ma"] = df["z_ma"].abs() > 2

# [MODIFIKASI] Membandingkan jumlah anomali
print("Anomali data asli:",
      df["anomali_asli"].sum())
print("Anomali setelah moving average:",
      df["anomali_ma"].sum())

# --- Tahap 3: Simpan ke File CSV & Status Penyimpanan ---
try:
    # Kondisional status penyimpanan CSV
    df["status_penyimpanan_csv"] = "Tersimpan"
    # [MODIFIKASI] Kolom suhu_ma, z_ma, dan anomali_ma ikut disimpan ke CSV
    kolom_tabel = ["waktu", "suhu", "zscore", "status_anomali",
                   "suhu_ma", "z_ma", "anomali_ma",
                   "status_penyimpanan_csv"]
    df.to_csv("data_suhu.csv", columns=kolom_tabel, index=False)
    simpan_sukses = True
except Exception as e:
    df["status_penyimpanan_csv"] = f"Gagal ({e})"
    simpan_sukses = False

if simpan_sukses:
    print("\n✓ Status: Seluruh data pengamatan (20 data) berhasil disimpan ke 'data_suhu.csv'")
else:
    print("\n✗ Status: Terjadi kesalahan saat menyimpan ke CSV")

# Ringkasan Statistik
print(f"\n--- Ringkasan Statistik ---")
print(f"Rata-rata : {rata2:.2f} °C")
print(f"Std deviasi : {std:.2f}")
print(f"Minimum : {df['suhu'].min():.2f} °C")
print(f"Maksimum : {df['suhu'].max():.2f} °C")
print(f"Total Sampel : {len(df)}")
print(f"Jumlah Anomali: {df['anomali'].sum()}\n")

# --- Tahap 4: Visualisasi data ---
plt.figure(figsize=(10, 4))
plt.plot(df.index, df["suhu"], marker="o", label="Suhu")

# [MODIFIKASI] Menampilkan garis moving average pada grafik
plt.plot(df.index, df["suhu_ma"], linestyle="-", label="Moving Average (5)")

anom = df[df["anomali"]]
plt.scatter(anom.index, anom["suhu"], color="red", s=100, label="Anomali", zorder=3)

plt.axhline(rata2, color="gray", linestyle="--", label="Rata-rata")
plt.xlabel("Urutan data")
plt.ylabel("Suhu (°C)")
plt.title("Data Suhu dan Deteksi Anomali")
plt.legend()
plt.tight_layout()
plt.savefig("grafik_suhu.png")
plt.show()
```
<h2>Penjelasan</h2>

Modifikasi dilakukan dengan menambahkan perhitungan moving average menggunakan jendela lima data sebelum menghitung z-score. Tujuannya adalah mengurangi fluktuasi pada data suhu sehingga proses deteksi anomali dapat dilakukan pada data yang lebih halus.
```cpp
# Menghitung moving average dengan jendela 5 data
df["suhu_ma"] = df["suhu"].rolling(
    window=5, min_periods=1
).mean()

# Deteksi anomali pada data suhu asli
rata2_asli = df["suhu"].mean()
std_asli = df["suhu"].std()
df["z_asli"] = (df["suhu"] - rata2_asli) / std_asli
df["anomali_asli"] = df["z_asli"].abs() > 2

# Deteksi anomali pada data hasil moving average
rata2_ma = df["suhu_ma"].mean()
std_ma = df["suhu_ma"].std()
df["z_ma"] = (df["suhu_ma"] - rata2_ma) / std_ma
df["anomali_ma"] = df["z_ma"].abs() > 2

# Membandingkan jumlah anomali
print("Anomali data asli:",
      df["anomali_asli"].sum())
print("Anomali setelah moving average:",
      df["anomali_ma"].sum())
```

Baris ```rolling(window=5, min_periods=1).mean()``` menghitung rata-rata bergerak dari lima data suhu terakhir, sedangkan ```min_periods=1``` memungkinkan perhitungan tetap dilakukan meskipun jumlah data pada awal pengambilan belum mencapai lima. Selanjutnya, program menghitung rata-rata, simpangan baku, dan z-score pada data asli serta data hasil moving average. Data dengan nilai absolut z-score lebih dari 2 ditandai sebagai anomali. Terakhir, jumlah anomali dari kedua metode ditampilkan agar hasilnya dapat dibandingkan. 
