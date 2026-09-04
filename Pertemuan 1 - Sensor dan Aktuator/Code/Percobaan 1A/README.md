<h1>Praktikum Percobaan 1A</h1>
<h3>Tujuan Praktikum</h3>
Memahami cara mikrokontroler melakukan akuisisi data suhu dan kelembaban dari sensor suhu serta mengetahui proses pembacaan, pengecekan, dan penampilan data sensor melalui Serial Monitor. 
<hr>
<h3>Penjelasan Code</h3>

```cpp
#include <DHT.h>
```
Digunakan untuk memasukkan library DHT agar mikrokontroler dapat berkomunikasi dan membaca data dari sensor DHT.
```cpp
#define DHTPIN 4
#define DHTTYPE DHT11
```
DHTPIN 4 menentukan GPIO 4 sebagai pin data sensor. DHTTYPE DHT11 menentukan jenis sensor yang digunakan adalah DHT11.
```cpp
DHT dht(DHTPIN, DHTTYPE);
```
Membuat objek dht berdasarkan pin dan jenis sensor yang telah ditentukan sehingga fungsi-fungsi dari library DHT dapat digunakan.
```cpp
Serial.begin(115200);
dht.begin();
Serial.println("Memulai akuisisi data sensor DHT22...");
```
Mengatur komunikasi dengan Serial Monitor pada baud rate 115200, inisialisasi sensor dengan dht.begin(). Kemudian menampilkan pesan awal pada Serial Monitor dengan Serial.println.
```cpp
float kelembaban = dht.readHumidity();
float suhu = dht.readTemperature();
```
Membuat variabel kelembaban dan suhu untuk membaca nilai dari sensor kemudian menyimpannya di variabel tersebut.
```cpp
if (isnan(kelembaban) || isnan(suhu)) {
  Serial.println("Gagal membaca data dari sensor DHT22!");
}
```
isnan() digunakan untuk memeriksa apakah hasil pembacaan sensor berupa NaN atau tidak valid. Jika salah satu pembacaan tidak valid, program menampilkan pesan gagal.
```cpp
else {
Serial.print("Suhu: ");
Serial.print(suhu);
Serial.print(" °C, Kelembaban: ");
Serial.print(kelembaban);
Serial.println(" %");
}
```
Jika pembacaan berhasil, nilai suhu dan kelembaban ditampilkan pada Serial Monitor beserta satuannya.
```cpp
delay(2000);
```
Memberikan jeda selama 2 detik sebelum program melakukan pembacaan sensor berikutnya.
<hr>
<h3>Pertanyaan Praktikum Modifikasi Program</h3>
Modifikasi program agar data suhu dan kelembaban dirata-ratakan dari 5 kali pembacaan sebelum ditampilkan, dan berikan penjelasan di setiap baris kode yang ditambahkan dalam bentuk README.md!

```cpp
#include <DHT.h>

#define DHTPIN 4
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);
  dht.begin();
  Serial.println("Memulai akuisisi data sensor DHT22...");
}

void loop() {
  // Variabel untuk menyimpan total suhu dari 5 pembacaan
  float totalSuhu = 0;
  
   // Variabel untuk menyimpan total kelembaban dari 5 pembacaan
  float totalKelembaban = 0;
  
  // Menentukan jumlah pembacaan yang dilakukan, yaitu 5 kali
  int jumlahPembacaan = 5; 

  for (int i = 0; i < jumlahPembacaan; i++) {  // Mengulang proses pembacaan sensor sebanyak 5 kali

    float kelembaban = dht.readHumidity();
    float suhu = dht.readTemperature();

    if (isnan(kelembaban) || isnan(suhu)) {
      Serial.println("Gagal membaca data dari sensor DHT22!");
      delay(2000);
      return;
    }

    totalSuhu += suhu; // Menambahkan hasil suhu ke total suhu
    totalKelembaban += kelembaban; //Menambahkan hasil kelembaban ke total kelembaban

    delay(2000); //Memberikan jeda 2 detik sebelum pembacaan berikutnya
  }

  //Menghitung rata-rata suhu dari 5 pembacaan
  float rataSuhu = totalSuhu / jumlahPembacaan;

   //Menghitung rata-rata kelembaban dari 5 pembacaan
  float rataKelembaban = totalKelembaban / jumlahPembacaan;

  Serial.print("Rata-rata Suhu: ");
  Serial.print(rataSuhu);
  Serial.print(" °C, Rata-rata Kelembaban: ");
  Serial.print(rataKelembaban);
  Serial.println(" %");
}
```
<h3>Penjelasan</h3>

```cpp
floa totalSuhu = 0;
float totalKelembaban = 0;
```
Digunakan untuk menyimpan jumlah nilai suhu an kelembaban dari seluruh pembacaan yang dilakukan.
```cpp
int jumlahPembacaan = 5;
```
Menentukan bahwa sensor akan melakukan pembacaan sebanyak 5 kali sebelum nilai rata-rata ditampilkan.
```cpp
for (int i = o; i < jumlahPembacaan; i++) {
```
digunakan untuk mengulang proses pembacaan sensor sebanyak 5 kali.
```cpp
totalSuhu += suhu;
totalKelembaban += kelembaban;
```
Setiap nilai suhu dan kelembaban yang berhasil dibaca ditambahkan ke masing-masing total untuk digunakan dalam perhitungan rata-rata.
```cpp
float rataSuhu = totalSuhu / jumlahPembacaan;
float rataKelembaban = totalKelembaban / jumlahPembacaan;
```
Menghitung nilai rata-rata suhu dan kelembaban dengan membagi total dari 5 pembacaan dengan jumlah pembacaan.
```cpp
Serial.print("Rata-rata Suhu: ");
Serial.print(rataSuhu);
Serial.print(" °C, Rata-rata Kelembaban: ");
Serial.print(rataKelembaban);
Serial.println(" %");
```
Menampilkan nilai rata-rata suhu dan kelembaban setelah kelima pembacaan selesai dilakukan.
<hr>
