<h1>Praktikum Percobaan </h1>
<h2>Tujuan Praktikum</h2>
Percobaan 3A dilakukan untuk mengimplementasikan komunikasi data antara ESP8266 dan server menggunakan protokol HTTP dengan metode POST. ESP8266 terlebih dahulu terhubung ke jaringan WiFi, kemudian data suhu dan kelembaban dibuat dalam format JSON. Data tersebut dikirimkan ke server secara berkala setiap 10 detik. Hasil pengiriman diamati melalui Serial Monitor dengan melihat kode respons HTTP serta data yang dikembalikan oleh server. 
<hr>
<h2>Penjelasan Code</h2>

```cpp
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <WiFiClientSecure.h>
#include <ArduinoJson.h>

const char* ssid = "Gejes";
const char* password = "cihuykandululeee";
const char* serverUrl = "https://httpbin.org/post";
```
Bagian ini digunakan untuk mempersiapkan library yang diperlukan dalam komunikasi WiFi, HTTP, HTTPS, dan JSON. Variabel ```ssid``` dan ```password``` digunakan untuk menghubungkan ESP8266 ke jaringan WiFi, sedangkan ```serverUrl``` menentukan alamat server tujuan pengiriman data.

```cpp
void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  
  Serial.print("Menghubungkan ke WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  
  Serial.println();
  Serial.println("WiFi berhasil terhubung!");
}
```
Bagian ```setup()``` digunakan untuk melakukan inisialisasi Serial Monitor dan menghubungkan ESP8266 ke jaringan WiFi. Looping ```while``` membuat program menunggu hingga ESP8266 berhasil terhubung ke WiFi sebelum melanjutkan proses.

```cpp
WiFiClientSecure client;
client.setInsecure();

HTTPClient http;
http.begin(client, serverUrl);
http.addHeader("Content-Type", "application/json");
```
Bagian ini menyiapkan koneksi HTTPS menggunakan ```WiFiClientSecure```. Perintah ```setInsecure()``` digunakan untuk mengabaikan verifikasi sertifikat SSL dalam pengujian. Lalu, ```HTTPClient``` digunakan untuk melakukan komunikasi HTTP dan header ditentukan bahwa data yang dikirim memiliki format JSON.

```cpp
JsonDocument doc;
doc["suhu"] = 28.5;
doc["kelembaban"] = 65.0;

String requestBody;
serializeJson(doc, requestBody);
```
Bagian ini digunakan untuk membuat data dalam format JSON. Data yang dikirim terdiri dari nilai suhu dan kelembaban. Fungsi ```serializeJson()``` kemudian mengubah objek JSON menjadi bentuk string agar dapat dikirim melalui HTTP.

```cpp
int httpResponseCode = http.POST(requestBody);

if (httpResponseCode > 0) {
  Serial.print("Kode Response HTTP: ");
  Serial.println(httpResponseCode);
  Serial.println("Isi Response:");
  Serial.println(http.getString());
} else {
  Serial.print("Pengiriman gagal, kode error: ");
  Serial.println(httpResponseCode);
}
```
Bagian ini digunakan untuk mengirim data JSON ke server menggunakan metode HTTP POST. Setelah pengiriman, program memeriksa kode respons dari server. jika pengiriman berhasil, kode dan isi respons ditampilkan pada Serial Monitor. Jika gagal, program menampilkan kode error.

```cpp
http.end();

delay(10000);
```
Perintah ```http.end()``` digunakan untuk mengakhiri koneksi HTTP setelah proses pengiriman selesai. Progarm kemudian berhenti selama 10 detik sebelum mengulangi proses pengiriman data berikutnya.
<hr>
<h2>Pertanyaan Praktikum Modifikasi Program</h2>
Modifikasi program agar ESP32 dapat mengirimkan data tambahan berupa waktu
(dalam milidetik sejak dinyalakan menggunakan millis()) ke dalam JSON yang dikirim,
dan berikan penjelasan di setiap baris kode yang ditambahkan dalam bentuk
README.md

```cpp
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <WiFiClientSecure.h>
#include <ArduinoJson.h>

const char* ssid     = "Gejes";
const char* password = "cihuykandululeee";
const char* serverUrl = "https://httpbin.org/post";

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  
  Serial.print("Menghubungkan ke WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();
  Serial.println("WiFi berhasil terhubung!");
}

void loop() {
  if (WiFi.status() == WL_CONNECTED) {
    
    WiFiClientSecure client;
    client.setInsecure();
    
    HTTPClient http;

    http.begin(client, serverUrl);
    http.addHeader("Content-Type", "application/json");
    
    JsonDocument doc;
    doc["timer"] = millis() / 1000; // ini buat nambahin timer
    doc["suhu"] = 28.5;         
    doc["kelembaban"] = 65.0; 
    
    String requestBody;
    serializeJson(doc, requestBody);
    
    Serial.print("Mengirim data: ");
    Serial.println(requestBody);
    
    int httpResponseCode = http.POST(requestBody);
    
    if (httpResponseCode > 0) {
      Serial.print("Kode Response HTTP: ");
      Serial.println(httpResponseCode);
      Serial.println("Isi Response:");
      Serial.println(http.getString());
    } else {
      Serial.print("Pengiriman gagal, kode error: ");
      Serial.println(httpResponseCode);
    }
    
    http.end();
  }
  
  delay(10000);
}
```
<h2>Penjelasan</h2>

```cpp
JsonDocument doc;
doc["timer"] = millis();
doc["suhu"] = 28.5;
doc["kelembaban"] = 65.0;
```
Baris ```doc["timer"] = millis();``` ditambahkan untuk memasukkan waktu sejak ESP8266 mulai dijalankan ke dalam data JSON. Fungsi ```millis()``` menghasilkan waktu dalam satuan milidetik, sehingga nilai tersebut dapat dikirim bersama data suhu dan kelembaban ke server. Jika ingin membuatnya menjadi detik, cukup bagi dengan 1000.
<hr>
