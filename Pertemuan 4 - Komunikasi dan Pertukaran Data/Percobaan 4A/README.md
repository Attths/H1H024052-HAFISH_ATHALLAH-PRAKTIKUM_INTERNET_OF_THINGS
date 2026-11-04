<h1>Praktikum Percobaan </h1>
<h2>Tujuan Praktikum</h2>
Percobaan 4A bertujuan untuk memahami dan mengimplementasikan mekanisme subscribe pada protokol MQTT serta proses deserialisasi data JSON yang diterima untuk mengendalikan aktuator. Pada percobaan ini, ESP8266 menerima pesan berupa perintah dari topic MQTT, kemudian mengubah data JSON tersebut menjadi nilai yang dapat diproses oleh program. Perintah yang diterima digunakan untuk mengatur kondisi LED sebagai aktuator, baik menyala maupun mati.
 
<hr>
<h2>Penjelasan Code</h2>

```cpp
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>

// Konfigurasi WiFi
const char* ssid = "Gejes";
const char* password = "Adalahpokoknya";

// Konfigurasi MQTT
const char* mqttServer = "broker.hivemq.com";
const int mqttPort = 1883;
const char* topicPerintah = "unsoed/tk245004/kelompok1/iotmodul4a";

// Konfigurasi LED
const int ledPin = 2;

// Membuat objek WiFi dan MQTT
WiFiClient espClient;
PubSubClient client(espClient);
```
Bagian ini mengimpor library utama untuk komunikasi wiFi (```ESP8266WiFi.h```), protokol MQTT (```PubSubClient.h```). dan pemrosesan JSON (```ArduinoJson.h```). Selain itu, didefinisikan variabel global untuk kredensial WiFi, alamat broker MQTT (HiveMQ), topik MQTT target, penentuan pin aktuator (GPIO2/D4), serta pembentukan instans objek ```espClient``` dan ```client```.

```cpp
void callback(char* topic, byte* payload, unsigned int length) {
  String pesan;
  for (unsigned int i = 0; i < length; i++) {
    pesan += (char)payload[i];
  }

  Serial.print("Pesan diterima [");
  Serial.print(topic);
  Serial.print("]: ");
  Serial.println(pesan);

  JsonDocument doc;
  DeserializationError error = deserializeJson(doc, pesan);

  if (error) {
    Serial.print("Gagal parsing JSON: ");
    Serial.println(error.c_str());
    return;
  }

  const char* perintah = doc["perintah"];

  if (String(perintah) == "ON") {
    digitalWrite(ledPin, HIGH);
    Serial.println("Aktuator: ON");
  } else if (String(perintah) == "OFF") {
    digitalWrite(ledPin, LOW);
    Serial.println("Aktuator: OFF");
  }
}
```
Fungsi ```callback``` dieksekusi secara otomatis setiap kali ada pesan baru dari broker MQTT pada topik yang diikuti. Data payload mentah diubah menjadi ```String```, lalu didekode menggunakan ```deserializeJson()```. Jika data JSON valid, nilai dari kunci ```"perintah"``` diekstrak untuk mengendalikan kondisi logika pin LED (```HIGH``` untuk menyalakan, ```LOW``` untuk mematikan).

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
Fungsi ini bertugas menginisalisasi proses koneksi jaringan WiFi menggunakan kredensial ```ssid``` dan ```password```. Perulangan ```while``` menahan eksekusi program (dengan jeda 500 ms) hingga ESP8266 berhasil memperoleh status ```WL_CONNECTED```.

```cpp
void hubungkanMQTT() {
  while (!client.connected()) {
    Serial.print("Menghubungkan ke broker MQTT...");
    String clientId = "ESP8266Client-" + String(random(0xffff), HEX);

    if (client.connect(clientId.c_str())) {
      Serial.println("berhasil terhubung!");
      client.subscribe(topicPerintah);
      Serial.print("Subscribe ke topic: ");
      Serial.println(topicPerintah);
    } else {
      Serial.print("gagal, rc=");
      Serial.print(client.state());
      Serial.println(" coba lagi dalam 2 detik");
      delay(2000);
    }
  }
}
```
Fungsi ini memastikan ESP8266 terhubung ke broker MQTT. Jika belum terhubung, ID klien acak (```clientId```) dibuat untuk menghindari konflik ID pada broker. Setelah berhasil melakukan koneksi via ```client.connect()```, modul akan otomatis melakukan subscribe ke ```topicPerintah``` agar dapat menerima perintah masuk. Jika gagal, sistem akan mencoba kembali setiap 2 detik.

```cpp
void setup() {
  Serial.begin(115200);

  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);

  hubungkanWiFi();
  client.setServer(mqttServer, mqttPort);
  client.setCallback(callback);
}
```
Fungsi ```setup()``` berjalan sekali saat perangkat pertama kali dinyalakan. Kode ini mengaktifkan komunikasi Serial Monitor pada bitrate 115200 bps, mengonfigurasi ```ledPin``` sebagai ```OUTPUT```, memanggil ```hubungkanWiFi()```, serta mengatur alamat server MQTT beserta fungsi pemicu ```callback```-nya.

```cpp
void loop() {
  if (!client.connected()) {
    hubungkanMQTT();
  }

  client.loop();
}
```
Fungsi ```loop()``` berjalan terus-menerus. Bagian ini melakukan pemeriksaan status koneksi MQTT secara berkala (dan memanggil reconnection jika terputus), serta mengontrol alur lalu lintas data MQTT masuk/keluar dan menjaga keep-alive broker melalui ```client.loop()```.

<hr>

<h2>Pertanyaan Praktikum Modifikasi Program</h2>
Modifikasi program agar data JSON yang diterima juga memuat nilai intensitas
(misalnya {"perintah": "ON", "intensitas": 200}) yang digunakan untuk mengatur
kecerahan LED menggunakan PWM (analogWrite/ledcWrite), dan berikan penjelasan di
setiap baris kode yang ditambahkan dalam bentuk README.md

```cpp
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>

// Konfigurasi WiFi
const char* ssid = "Gejes";
const char* password = "Adalahpokoknya";

// Konfigurasi MQTT
const char* mqttServer = "broker.hivemq.com";
const int mqttPort = 1883;
const char* topicPerintah = "unsoed/tk245004/kelompok1/iotmodul4a";

// Konfigurasi LED
const int ledPin = 2;

// Membuat objek WiFi dan MQTT
WiFiClient espClient;
PubSubClient client(espClient);

// Fungsi callback dipanggil otomatis ketika ada pesan baru
void callback(char* topic, byte* payload, unsigned int length) {
  String pesan;

  // Mengubah payload menjadi String
  for (unsigned int i = 0; i < length; i++) {
    pesan += (char)payload[i];
  }

  Serial.print("Pesan diterima [");
  Serial.print(topic);
  Serial.print("]: ");
  Serial.println(pesan);

  // Deserialisasi data JSON
  JsonDocument doc;
  DeserializationError error = deserializeJson(doc, pesan);

  // Mengecek hasil parsing JSON
  if (error) {
    Serial.print("Gagal parsing JSON: ");
    Serial.println(error.c_str());
    return;
  }

  // Mengambil nilai perintah dari JSON
  const char* perintah = doc["perintah"];

  // [MODIFIKASI] Mengambil nilai intensitas dari JSON
  // Jika key "intensitas" tidak ada, nilai default = 255 (terang penuh)
  int intensitas = doc["intensitas"] | 255;

  // [MODIFIKASI] Membatasi nilai intensitas agar tetap di rentang 0-255
  intensitas = constrain(intensitas, 0, 255);

  // Mengecek apakah key "perintah" ada pada JSON
  if (perintah == nullptr) {
    Serial.println("Key 'perintah' tidak ditemukan pada JSON");
    return;
  }

  // Mengontrol LED
  if (String(perintah) == "ON") {
    // [MODIFIKASI] Mengatur kecerahan LED menggunakan PWM
    analogWrite(ledPin, intensitas);
    Serial.print("Aktuator: ON, intensitas: ");
    Serial.println(intensitas);

  } else if (String(perintah) == "OFF") {
    // [MODIFIKASI] PWM = 0 sehingga LED mati
    analogWrite(ledPin, 0);
    Serial.println("Aktuator: OFF");
  }
}

// Fungsi untuk menghubungkan ESP8266 ke WiFi
void hubungkanWiFi() {
  WiFi.begin(ssid, password);

  Serial.print("Menghubungkan ke WiFi");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nWiFi berhasil terhubung!");
}

// Fungsi untuk menghubungkan ESP8266 ke broker MQTT
void hubungkanMQTT() {
  while (!client.connected()) {
    Serial.print("Menghubungkan ke broker MQTT...");

    // Membuat ID client secara acak
    String clientId = "ESP8266Client-" + String(random(0xffff), HEX);

    if (client.connect(clientId.c_str())) {
      Serial.println("berhasil terhubung!");

      // Subscribe ke topic setelah berhasil terhubung
      client.subscribe(topicPerintah);

      Serial.print("Subscribe ke topic: ");
      Serial.println(topicPerintah);

    } else {
      Serial.print("gagal, rc=");
      Serial.print(client.state());
      Serial.println(" coba lagi dalam 2 detik");

      delay(2000);
    }
  }
}

// Fungsi setup dijalankan sekali
void setup() {
  Serial.begin(115200);

  // Mengatur LED D4 sebagai output
  pinMode(ledPin, OUTPUT);

  // [MODIFIKASI] Mengatur rentang PWM ESP8266 menjadi 0-255
  // (default ESP8266 adalah 0-1023)
  analogWriteRange(255);

  analogWrite(ledPin, 0);

  // Menghubungkan ESP8266 ke WiFi
  hubungkanWiFi();

  // Mengatur server MQTT
  client.setServer(mqttServer, mqttPort);

  // Mendaftarkan fungsi callback
  client.setCallback(callback);
}

// Fungsi loop dijalankan terus-menerus
void loop() {
  // Mengecek koneksi MQTT
  if (!client.connected()) {
    hubungkanMQTT();
  }

  // Memproses pesan MQTT yang masuk
  client.loop();
}

```
<h2>Penjelasan</h2>

Data yang dikirim dapat dibuat seperti ini:
```cpp
{
    “Perintah”: “ON”,
    “Intensitas”: 200,
}
```

Kemudian program dapat mengambil nilai intensitas dari JSON dan menggunakannya untuk mengatur kecerahan LED.

```cpp
const char* perintah = doc["perintah"];
int intensitas = doc["intensitas"];

if (String(perintah) == "ON") {
  analogWrite(ledPin, intensitas);
  Serial.println("Aktuator: ON");
} else if (String(perintah) == "OFF") {
  analogWrite(ledPin, 0);
  Serial.println("Aktuator: OFF");
}

```
Dengan demikian, nilai intensitas dapat digunakan untuk menentukan tingkat kecerahan LED, sedangkan perintah “OFF” mengatur nilai PWM menjadi 0 sehingga LED mati.
