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
