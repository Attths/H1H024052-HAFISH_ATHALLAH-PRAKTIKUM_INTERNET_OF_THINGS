<h1>Praktikum Percobaan </h1>
<h2>Tujuan Praktikum</h2>
Percobaan 5A bertujuan untuk memahami dan mengimplementasikan proses pengolahan data sensor pada sisi perangkat (edge processing) menggunakan ESP8266 dan data suhu simulasi (dummy). Pada percobaan ini, data suhu dibuat menggunakan fungsi yang mensimulasikan perubahan suhu secara bertahan, fluktuasi acak, dan lonjakan suhu sebagai anomali. Data tersebut kemudian diolah menggunakan filter moving average untuk mengurangi fluktuasi, serta digunakan untuk menghitung nilai suhu minimum dan maksimum. Hasil pengolahan diamati melalui Serial Monitor untuk memahami proses penyaringan dan analisis data pada sisi perangkat (edge processing).
 
<hr>
<h2>Penjelasan Code</h2>

```cpp
const int N = 5;          // jumlah sampel moving average
float buffer[N];          // menyimpan N pembacaan terakhir
int indeks = 0;           // posisi penulisan berikutnya pada buffer
int jumlah = 0;           // jumlah sampel yang sudah terisi

float suhuMin = 1000.0;
float suhuMax = -1000.0;
```
Bagian ini mendefinisikan variabel global untuk filter moving average dan pelacakan nilai ekstrem. ```N``` adalah ukuran jendela (5 sampel terakhir), ```buffer[]``` menyimpan sampel tersebut, ```indeks``` menunjuk posisi penulisan berikutnya, dan ```jumlah``` mencatat berapa sampel yang sudah terisi. Variabel ```suhuMin``` dan ```suhuMax``` sengaja diisi nilai ekstrem (1000 dan -1000) agar pembacaan pertama otomatis menggantikan keduanya.

```cpp
float bacaSuhuMock() {
    static unsigned long jumlahBaca = 0;
    jumlahBaca++;

    float suhuDasar = 28.0 + 2.0 * sin(jumlahBaca * 0.08);
    float noise = random(-5, 6) * 0.1;
    float suhu = suhuDasar + noise;

    if (jumlahBaca % 25 == 0) {
        suhu += 8.0;
    }

    return suhu;
}
```
Fungsi ```bacaSuhuMock()``` menggantikan sensor DHT22 dengan data simulasi. Variabel ```static jumlahBaca``` mempertahankan nilainya di antara pemanggilan fungsi, sehingga berfungsi sebagai penghitung pembacaan. Suhu dasar dibuat berosilasi pelan di sekitar 28 °C (rentang 26–30 °C) menggunakan fungsi ```sin()```. Lalu ditambahkan ```noise``` acak antara -0.5 sampai +0.5 °C untuk meniru fluktuasi sensor. Pada setiap pembacaan kelipatan 25, suhu ditambah 8 °C untuk mensimulasikan anomali berupa lonjakan mendadak.

```cpp
float hitungMovingAverage(float nilaiBaru) {
  buffer[indeks] = nilaiBaru;
  indeks = (indeks + 1) % N;
  if (jumlah < N) jumlah++;

  float total = 0;
  for (int i = 0; i < jumlah; i++) {
    total += buffer[i];
  }
  return total / jumlah;
}
```
Fungsi ini menerapkan filter moving average dengan struktur circular buffer. Nilai baru ditulis ke ```buffer[indeks]```, lalu indeks maju satu posisi dan kembali ke 0 setelah mencapai ```N``` berkat operator modulo (%), sehingga data terlama otomatis tertimpa. Variabel jumlah memastikan pembagian hanya dilakukan terhadap sampel yang sudah terisi, jadi rata-rata di awal program tidak terdistorsi oleh elemen kosong. Hasilnya adalah rata-rata dari maksimal 5 pembacaan terakhir, yang meredam noise dan lonjakan sesaat.

```cpp
void setup() {
  Serial.begin(115200);
  randomSeed(analogRead(0));
}
```
Fungsi setup() dijalankan sekali saat program dimulai. Serial Monitor diaktifkan dengan baud rate 115200, dan ```randomSeed(analogRead(0))``` menginisialisasi generator angka acak dengan pembacaan pin analog yang tidak terhubung. Dengan begitu, pola noise berbeda setiap kali program dijalankan.

```cpp
void loop() {
  float waktuPengiriman = millis() / 1000.0;

  float suhu = bacaSuhuMock();

  if (isnan(suhu)) {
    Serial.println("Pembacaan gagal, data diabaikan");
  } else {
    float suhuFilter = hitungMovingAverage(suhu);

    if (suhu < suhuMin) suhuMin = suhu;
    if (suhu > suhuMax) suhuMax = suhu;

    Serial.print("Waktu: ");
    Serial.print(waktuPengiriman, 1);
    Serial.print("s | Mentah: ");
    Serial.print(suhu);
    Serial.print(" | Filter: ");
    Serial.print(suhuFilter);
    Serial.print(" | Min: ");
    Serial.print(suhuMin);
    Serial.print(" | Max: ");
    Serial.println(suhuMax);
  }

  delay(2000);
}
```
Fungsi ```loop()``` berjalan terus-menerus dan memproses satu pembacaan suhu setiap siklus. Suhu diambil dari ```bacaSuhuMock()```, lalu dimasukkan ke ```hitungMovingAverage()``` untuk mendapatkan nilai terfilter. Nilai mentah dibandingkan dengan ```suhuMin``` dan ```suhuMax``` untuk memperbarui rekor terendah dan tertinggi. Pengecekan ```isnan()``` mengabaikan pembacaan yang gagal. Waktu (```millis() / 1000.0```), suhu mentah, suhu terfilter, min, dan max dicetak ke Serial Monitor, lalu ```delay(2000)``` mengulang proses setiap 2 detik.

<hr>

<h2>Pertanyaan Praktikum Modifikasi Program</h2>
Modifikasi program agar ESP32 juga menghitung simpangan baku dari data pada
buffer, dan berikan penjelasan di setiap baris kode yang ditambahkan dalam bentuk
README.md

```cpp
const int N = 5;          // jumlah sampel moving average
float buffer[N];          // menyimpan N pembacaan terakhir
int indeks = 0;           // posisi penulisan berikutnya pada buffer
int jumlah = 0;           // jumlah sampel yang sudah terisi

float suhuMin = 1000.0;
float suhuMax = -1000.0;

// Mock function pengganti sensor DHT22
float bacaSuhuMock() {
    static unsigned long jumlahBaca = 0;
    jumlahBaca++;

    // Suhu dasar berubah secara bertahap
    float suhuDasar = 28.0 + 2.0 * sin(jumlahBaca * 0.08);

    // Noise kecil untuk mensimulasikan fluktuasi sensor
    float noise = random(-5, 6) * 0.1;

    float suhu = suhuDasar + noise;

    // Lonjakan suhu setiap 25 pembacaan untuk simulasi anomali
    if (jumlahBaca % 25 == 0) {
        suhu += 8.0;
    }

    return suhu;
}

float hitungMovingAverage(float nilaiBaru) {
  buffer[indeks] = nilaiBaru;
  indeks = (indeks + 1) % N;
  if (jumlah < N) jumlah++;

  float total = 0;
  for (int i = 0; i < jumlah; i++) {
    total += buffer[i];
  }
  return total / jumlah;
}

// [MODIFIKASI] Fungsi untuk menghitung simpangan baku data pada buffer
float hitungSimpanganBaku() {
  float total = 0;
  float rataRata = 0;
  float jumlahKuadrat = 0;

  // Menjumlahkan seluruh data yang sudah terisi pada buffer
  for (int i = 0; i < jumlah; i++) {
    total += buffer[i];
  }

  // Menghitung rata-rata data pada buffer
  rataRata = total / jumlah;

  // Menjumlahkan kuadrat selisih setiap data terhadap rata-rata
  for (int i = 0; i < jumlah; i++) {
    float selisih = buffer[i] - rataRata;
    jumlahKuadrat += selisih * selisih;
  }

  // Akar dari rata-rata kuadrat selisih = simpangan baku
  return
```
<h2>Penjelasan</h2>

Fungsi untuk menghitung simpangan bakui:
```cpp
float hitungSimpanganBaku() {
  float total = 0;
  float rataRata = 0;
  float jumlahKuadrat = 0;

  for (int i = 0; i < jumlah; i++) {
    total += buffer[i];
  }
  rataRata = total / jumlah;

  for (int i = 0; i < jumlah; i++) {
    float selisih = buffer[i] - rataRata;
    jumlahKuadrat += selisih * selisih;
  }

  return sqrt(jumlahKuadrat / jumlah);
}

```

Kemudian menambahkan kode untuk menampilkan nilai simpangan baku di dalam loop(), tepat setelah float suhuFilter = hitungMovingAverage(suhu);.

```cpp
float simpanganBaku = hitungSimpanganBaku();

Serial.print(" | Std: ");
Serial.print(simpanganBaku);
```
