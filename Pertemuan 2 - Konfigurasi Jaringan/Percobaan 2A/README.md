<h1>Praktikum Percobaan 2A</h1>
<h2>Tujuan Praktikum</h2>
Percobaan 2A bertujuan untuk memahami dan mengimplementasikan ESP32 sebagai Station
(STA), yaitu sebagai perangkat yang terhubung ke jaringan WiFi yang sudah tersedia. ESP32
dikonfigurasi agar dapat terhubung menggunakan SSID dan password tertentu, kemudian
mengamati informasi jaringan seperti status koneksi, IP address, MAC address, dan kekuatan sinyal
(RSSI) melalui serial monitor.
<hr>
<h2>Penjelasan Kode</h2>

```cpp
#include <ESP8266WiFi.h>

const char* ssid = "Gejes gejes";
const char* password = "cihuykandululeee";

const int ledPin = 2;
```
Bagian ini digunakan untuk menyiapkan kebutuhan koneksi WiFi pada ESP8266. Library ESP8266WiFi menyediakan fungsi
untuk mengatur koneksi WiFi. Variabel ssid dan password digunakan untuk menyimpan nama dan kata sandi jaringan
WiFi yang akan dihubungkan. Selain itu, ledPin menentukan pin yang digunakan sebagai indikator LED, yaitu GPIO 2
atau D4 pada NodeMCU8266.

```cpp
Serial.begin(115200);

pinMode(ledPin, OUTPUT);
ddigitalWrite(ledPin, LOW);
```
Bagian ini mengatur Komunikasi serial yang diatur pada baud rate 115200 agar informasi dari ESP8266 dapat ditampilkan
pada Serial Monitor. Pin LED diatur sebagai output dan kondisi awal LED dibuat mati. LED nantinya digunakan sebagai
indikator apakah ESP8266 berhasil terhubung ke WiFi atau tidak.

```cpp
WiFi.mode(WIFI_STA);
WiFi.begin(ssid, password);
```
Bagian ini mengatur ESP8266 dalam mode Station (STA). Dalam mode ini, ESP8266 bertindak sebagai perangkat yang
bergabung ke jaringan WiFi yang sudah tersedia. Fungsi ```WiFi.begin()``` kemudian memulai proses koneksi menggunakan SSId dan
password yang telah ditentukan

```cpp
Serial.print("Menghubungkan ke WiFi");

while (WiFi.status() != WL_CONNECTED) {
  delay(500);
  Serial.print(".");
}
```
Bagian ini digunakan untuk mengunggu sampai ESP8266 berhasil terhubung ke WiFi. Program akan tersu memeriksa status koneksi melalui ```WiFi.status()```. Selama belum mendapatkan status WL_CONNECTED, ESP8266 akan menunggu selama 500 milidetik kemudian mencetak titik sebagai tanda bahwa proses koneksi masih berlangsung. Jika SSID atau password salah, bagian ini akan terus berjalan sehingga titik akan terus muncul dan program belum melanjutkan ke bagian berikutnya.

```cpp
Serial.println("WiFi berhasil terhubung!");

Serial.print("IP Address : ");
Serial.println(WiFi.localIP());

Serial.print("MAC Address : ");
Serial.println(WiFi.macAddress());

Serial.print("RSSI (dBm) : ");
Serial.println(WiFi.RSSI());
```
Bagian ini dijalankan setelah ESP8266 berhasil terhubung ke WiFi. Serial Monitor akan menampilkan informasi jaringan berupa status keberhasilan koneksi, alamat IP yang diberikan oleh jaringan, alamat MAC dari ESP8266, serta nilai RSSI yang menunjukkan kekuatan sinyal WiFi dalam satuan dBm. Nilai RSSI umumnya berupa angka negatif, dan nilai yang semakin mendekati 0 menunjukkan sinyal semakin kuat.

```cpp
digitalWrite(ledPin, HIGH);
```
Bagian ini mengatur LED menjadi menyala ketika koneksi WiFi berhasil. Dengan demikian, LED dapat digunakan sebagai indikator visual tanpa harus selalu melihat Serial Monitor.

```cpp
void loop() {
  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("Status: Terhubung");
  } else {
    Serial.println("Status: Terputus");
    digitalWrite(ledPin, LOW);
  }

  delay(5000);
}
```
Bagian loop() digunakan untuk memantau status koneksi WiFi secara berulang. Setiap lima detik, ESP8266 memeriksa apakah masih terhubung ke jaringan. Jika masih terhubung, Serial Monitor menampilkan "Status: Terhubung". Jika koneksi terputus, Serial Monitor menampilkan "Status: Terpututs" dan LED dimatikan sebagai indikator bahwa koneksi tidak tersedia.
<hr>
<h2>Pertanyaan Praktikum Modifikasi Kode</h2>

```cpp
#include <ESP8266WiFi.h>

const char* ssid = "NAMA_WIFI_ANDA";
const char* password = "PASSWORD_WIFI_ANDA";

const int ledPin = 2; // LED indikator status koneksi

void setup() {
  Serial.begin(115200);
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);

  // Set mode WiFi menjadi Station
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  Serial.print("Menghubungkan ke WiFi");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  // Jika berhasil terhubung
  Serial.println();
  Serial.println("WiFi berhasil terhubung!");

  Serial.print("IP Address : ");
  Serial.println(WiFi.localIP());

  Serial.print("MAC Address : ");
  Serial.println(WiFi.macAddress());

  Serial.print("RSSI (dBm) : ");
  Serial.println(WiFi.RSSI());

  digitalWrite(ledPin, HIGH);
}

void loop() {
  // Cek status koneksi setiap 5 detik
  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("Status: Terhubung");
    digitalWrite(ledPin, HIGH); // TAMBAHAN
  } else {
    Serial.println("Status: Terputus");
    digitalWrite(ledPin, LOW);

    // TAMBAHAN: mencoba menghubungkan kembali ke WiFi
    Serial.println("Mencoba menghubungkan kembali...");
    WiFi.reconnect();

    // TAMBAHAN: memberikan waktu untuk proses reconnect
    delay(1000);
  }

  delay(5000);
}
```
<h2>Penjelasan</h2>

```cpp
digitalWrite(ledPin, HIGH);
```
Menambahkan perintah menyalakan LED pada bagian kondisi ```if```. Dengan penambahan ini, LED dipastikan tetap menyala selama ESP8266 terhubung ke WiFi. Jadi status LED tidak hanya diatur ketika pertama kali berhasil terhubung pada ```setup()```, tetapi juga diperbarui setiap kali pemeriksaan koneksi dilakukan.

```cpp
Serial.println("Mencoba menghubungkan kembali...");
```
Pesan pada bagian ```else``` untuk informasi pada Serial Monitor bahwa ESP8266 mendeteksi koneksi terputus dan sedang mencoba melakukan koneksi ulang.

```cpp
WiFi.reconnect();
delay(1000);
```
Fungsi ini digunakan untuk memerintahkan ESP8266 mencoba menyambungkan kembali ke jaringan WiFi yang sebelumnya digunakan. Jadi ketika WiFi terputus, ESP8266 tidak hanya melaporkan status Terputus, tetapi juga melakukan proses reconnect. Setelah melakukan reconnect, diberikan Delay selama satu detik untuk memberikan waktu bagi ESP8266 dalam menjalankan proses reconnect. Setelah waktu tersebut, program akan melanjutkaan proses pemeriksaan status koneksi pada iterasi berikutnya.
<hr>
