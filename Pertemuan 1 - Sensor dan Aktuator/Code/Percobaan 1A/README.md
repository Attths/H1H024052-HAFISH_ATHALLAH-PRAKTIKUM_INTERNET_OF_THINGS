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
sht.begin();
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
