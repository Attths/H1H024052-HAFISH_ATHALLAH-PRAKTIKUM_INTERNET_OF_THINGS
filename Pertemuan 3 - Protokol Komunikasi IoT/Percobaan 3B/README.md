<h1>Praktikum Percobaan 3B</h1>
<h2>Tujuan Praktikum</h2>
Tujuan Percobaan 3B adalah memahami dan mengimplementasikan komunikasi data menggunakan protokol MQTT dengan pola publish-subscribe. Pada percobaan ini, ESP8266 dihubungkan ke broker MQTT, lalu mempublikasikan data sensor dalam format JSON secara berkala pada suatu topic, serta memverifikasi data yang dipublikasikan melalui aplikasi client MQTT yang melakukan subscribe pada topic tersebut.
<hr>
<h2>Penjelasan Kode</h2>

```cpp
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>

const char* ssid = "Gejes";
const char* password = "********";

const char* mqttServer = "broker.hivemq.com";
const int mqttPort = 1883;
const char* mqttTopic = "unsoed/tk245004/kelompok1/sensor";
```
Bagian ini mempersiapkan library untuk koneksi WiFi, komunikasi MQTT, dan pengolahan data JSON. Variabel ```mqttServer```, ```mqttPort```, dan ```mqttTopic``` digunakan untuk menentukan broker MQTT, port komunikasi, serta topic yang menjadi tujuan pengiriman data.

```cpp
WiFiClient espClient;
PubSubClient client(espClient);
```
Bagian ini membuat obje ```WiFiClient``` sebagai koneksi jaringan dan ```PubSubClient``` sebagai client MQTT yang digunakan ESP8266 untuk brekomunikasi dengan broker.

```cpp
void hubungkanWiFi() {
  WiFi.begin(ssid, password);

  Serial.print("Menghubungkan ke WiFi");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nWiFi berhasil terhubung!");
}
```
Fungs ```hubungkanWiFi()``` digunakan untuk menghubungkan ESP8266 ke jaringan WiFi. Program akan terus menunggu sampai koneksi berhasil, kemudian menampilkan statusnya ke monitor.

```cpp
void hubungkanMQTT() {
  while (!client.connected()) {
    Serial.print("Menghubungkan ke broker MQTT...");

    String clientId = "ESP32Client-" + String(random(0xffff), HEX);

    if (client.connect(clientId.c_str())) {
      Serial.println("berhasil terhubung!");
    } else {
      Serial.print("gagal, rc=");
      Serial.println(client.state());
      Serial.println("coba lagi dalam 2 detik");
      delay(2000);
    }
  }
}
```
Fungsi ```hubungkanMQTT()``` digunakan untuk menghubungkan perangkat dengan broker MQTT. Program membuat ```clientId``` secara acak dan akan terus mencoba melakukan koneksi apabila koneksi sebelumnya gagal. Status koneksi atau kode error ditampilkan pada Serial Monitor.

```cpp
void setup() {
  Serial.begin(115200);

  hubungkanWiFi();

  client.setServer(mqttServer, mqttPort);
}
```
Bagian ```setup()``` melakukan inisialisasi komunikasi Serial, menghubungkan ESP8266 ke WiFi, dan menentukan alamat serta port broker MQTT yang akan digunakan.

```cpp
if (!client.connected()) {
  hubungkanMQTT();
}

client.loop();
```

Bagian ini memastikan ESP8266 tetap terhubung dengan broker MQTT. Jika koneksi terputus, fungsi ```hubungkanMQTT()``` akan dipanggil kembali. Perintah ```cleint.loop()``` digunakan untuk mempertahankan dan memproses komunikasi MQTT selama program berjalan.

```cpp
JsonDocument doc;
doc["timer"] = millis() / 1000;
doc["suhu"] = 28.5;
doc["kelembaban"] = 65.0;

char buffer[128];
serializeJson(doc, buffer);
```

Bagian ini membuat data yang akan dikirim dalam format JSON. Data terdiri dari ```timer```, ```suhu```, dan ```kelembaban```. Fungsi ```serializeJson()``` mengubah objek JSON menjadi teks yang disimpan dalam ```buffer``` sehingga siap dipublikasikan melalui MQTT.

```cpp
client.publish(mqttTopic, buffer);

Serial.print("Data terkirim ke topic ");
Serial.print(mqttTopic);
Serial.print(": ");
Serial.println(buffer);
```
Bagian ini mengirimkan data JSON ke topic MQTT yang telah ditentukan menggunakan ```client.publish()```. Data yang dikirim kemudian ditampilkan pada Serial Monitor seagai informasi bahwa proses publikasi telah dilakukan.

```cpp
delay(5000);
```
program menunggu selama 5 detik setelah melakukan publikasi. Setelah itu, proses pada ```loop()``` akan dijalankan kembali sehingga data JSON dikirim secara berkala setiap 5 detik.
