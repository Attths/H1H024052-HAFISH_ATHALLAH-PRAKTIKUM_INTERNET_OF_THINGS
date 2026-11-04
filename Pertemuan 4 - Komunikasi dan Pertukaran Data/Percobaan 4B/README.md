<h1>Praktikum Percobaan </h1>
<h2>Tujuan Praktikum</h2>
Percobaan 4B bertujuan untuk mengimplementasikan pertukaran data dua arah antara perangkat IoT dan broker MQTT secara bersamaan (full duplex). ESP8266 digunakan untuk mengirimkan data suhu dari sensor DHT11 secara berkala melalui mekanisme publish, sekaligus menerima perintah kendali melalui mekanisme subscribe untuk mengatur LED. Percobaan ini juga bertujuan untuk memahami penerapan komunikasi non-blocking menggunakan millis(), sehingga proses pengiriman data sensor dan penerimaan perintah dapat berjalan tanpa saling mengganggu.
 
<hr>
<h2>Penjelasan Code</h2>

```cpp
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>
#include <DHT.h>

// Konfigurasi WiFi
const char* ssid = "Gejes";
const char* password = "Adalahpokoknya";

// Konfigurasi MQTT
const char* mqttServer = "broker.hivemq.com";
const int mqttPort = 1883;
const char* topicData = "unsoed/tk245004/kelompok1/datasensor";
const char* topicPerintah = "unsoed/tk245004/kelompok1/iotmodul4b";

// Konfigurasi Hardware Sensor & Aktuator
#define DHTPIN D2       // D2 = GPIO4
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

const int ledPin = D4;  // D4 = GPIO2
```
Bagian ini mengimpor semua pustaka yang dibutuhkan (WiFi, MQTT, ArduinoJson, dan Sensor DHT). Mendefinisikan kredensial jaringan WiFi, konfigurasi broker MQTT beserta dua topik terpisah (```topicData``` untuk mempublikasikan data suhu dan ```topicPerintah``` untuk menerima instruksi kendali LED), serta menginisialisasi pin hardware DHT11 pada pin D2 (GPIO4) dan LED pada pin D4 (GPIO2).

```cpp
WiFiClient espClient;
PubSubClient client(espClient);

// Waktu publish
unsigned long waktuTerakhirPublish = 0;
const long intervalPublish = 5000;
```
Bagian ini membuat instans objek ```espClient``` dan ```client``` untuk mengelola lalu lintas data network dan komunikasi MQTT. Variabel ```waktuTerakhirPublish``` dan ```intervalPublish``` digunakan sebagai penanda waktu berbasis fungsi ```millis()``` agar proses publish data terjadi secara berkala setiap 5000 ms (5 detik) tanpa perlu menghentikan alur program (non-blocking).

```cpp
void callback(char* topic, byte* payload, unsigned int length) {
  String pesan;
  for (unsigned int i = 0; i < length; i++) {
    pesan += (char)payload[i];
  }

  JsonDocument doc;
  if (deserializeJson(doc, pesan)) {
    Serial.println("Gagal membaca JSON!");
    return;
  }

  const char* perintah = doc["perintah"];

  if (String(perintah) == "ON") {
    digitalWrite(ledPin, HIGH);
    Serial.println("Perintah diterima -> Aktuator: ON");
  } else if (String(perintah) == "OFF") {
    digitalWrite(ledPin, LOW);
    Serial.println("Perintah diterima -> Aktuator: OFF");
  }
}
```
Fungsi ```callback``` dieksekusi secara otomatis setiap kali ada pesan masuk pada topik yang di-subscribe. Payload mentah diubah menjadi ```String```, lalu didekode menggunakan ```deserializeJson()```. Jika format JSON valid, nilai dari kunci ```"perintah"``` diekstrak untuk mengatur logika pin LED (HIGH untuk menyalakan, ```LOW``` untuk mematikan).

```cpp
void hubungkanWiFi() {
  WiFi.begin(ssid, password);
  Serial.print("Menghubungkan ke WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi berhasil terhubung!");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());
}

void hubungkanMQTT() {
  while (!client.connected()) {
    Serial.print("Menghubungkan ke MQTT...");
    String clientId = "ESP8266Client-" + String(random(0xffff), HEX);
    if (client.connect(clientId.c_str())) {
      Serial.println("berhasil!");
      client.subscribe(topicPerintah);
      Serial.print("Subscribe ke: ");
      Serial.println(topicPerintah);
    } else {
      Serial.print("Gagal, rc=");
      Serial.print(client.state());
      Serial.println(" - mencoba lagi...");
      delay(2000);
    }
  }
}
```
Fungsi ```hubungkanWiFi()``` mengaktifkan pencarian koneksi ke jaringan WiFi hingga berhasil tersambung. Fungsi ```hubungkanMQTT()``` melakukan koneksi ulang ke broker MQTT dengan Client ID acak jika koneksi terputus, lalu mendaftarkan perangkat (subscribe) ke ```topicPerintah``` segera setelah statusnya terhubung.

```cpp
void setup() {
  Serial.begin(115200);

  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);

  dht.begin();
  hubungkanWiFi();

  client.setServer(mqttServer, mqttPort);
  client.setCallback(callback);
}
```
Fungsi ```setup()``` berjalan sekali saat awal perangkat dinyalakan. Kode ini mengaktifkan komunikasi Serial Monitor pada baud rate 115200 bps, mengonfigurasi ```ledPin``` sebagai ```OUTPUT```, menginisialisasi sensor DHT11 (```dht.begin()```), menjalankan fungsi koneksi WiFi, serta mengatur server MQTT dan mendaftarkan fungsi ```callback```.

```cpp
void loop() {
  if (!client.connected()) {
    hubungkanMQTT();
  }
  client.loop();

  if (millis() - waktuTerakhirPublish > intervalPublish) {
    waktuTerakhirPublish = millis();
    float suhu = dht.readTemperature();

    if (!isnan(suhu)) {
      JsonDocument doc;
      doc["suhu"] = suhu;

      char buffer[128];
      serializeJson(doc, buffer);

      client.publish(topicData, buffer);
      Serial.print("Data terkirim: ");
      Serial.println(buffer);
    } else {
      Serial.println("Gagal membaca DHT11!");
    }
  }
}
```
Fungsi ```loop()``` menjalankan alur komunikasi dua arah (full duplex). ```client.loop()``` rutin memeriksa pesan masuk dari broker agar respons penerimaan perintah tetap cepat. Pengecekan ```millis() - waktuTerakhirPublish > intervalPublish``` memastikan pembacaan suhu DHT11 dan serialisasi ke format JSON untuk di-publish ke ```topicData``` hanya dieksekusi setiap 5 detik tanpa menghambat alur kerja penerimaan data (tanpa pemblokiran oleh ```delay()```).

<hr>

<h2>Pertanyaan Praktikum Modifikasi Program</h2>
Modifikasi program agar menambahkan satu topic perintah baru untuk mengendalikan
aktuator kedua (misalnya buzzer), dengan fungsi callback yang dapat membedakan topic
mana yang menerima pesan, dan berikan penjelasan di setiap baris kode yang
ditambahkan dalam bentuk README.md!

```cpp
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>
#include <DHT.h>

// =======================
// WiFi
// =======================
const char* ssid = "Gejes";
const char* password = "Adalahpokoknya";

// =======================
// MQTT
// =======================
const char* mqttServer = "broker.hivemq.com";
const int mqttPort = 1883;

const char* topicData = "unsoed/tk245004/kelompok1/datasensor";
const char* topicPerintah = "unsoed/tk245004/kelompok1/iotmodul4b";

// [MODIFIKASI] Topic baru untuk mengendalikan aktuator kedua (buzzer)
const char* topicBuzzer = "unsoed/tk245004/kelompok1/buzzer";

// =======================
// DHT11
// =======================
#define DHTPIN D2       // D2 = GPIO4
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);

// =======================
// LED
// =======================
const int ledPin = D4;  // D4 = GPIO2

// [MODIFIKASI] Pin buzzer sebagai aktuator kedua
const int buzzerPin = D5;  // D5 = GPIO14

// =======================
// MQTT Client
// =======================
WiFiClient espClient;
PubSubClient client(espClient);

// Waktu publish
unsigned long waktuTerakhirPublish = 0;
const long intervalPublish = 5000;

// =======================
// Callback MQTT
// =======================
void callback(char* topic, byte* payload, unsigned int length) {
  String pesan;

  for (unsigned int i = 0; i < length; i++) {
    pesan += (char)payload[i];
  }

  // Membaca JSON
  JsonDocument doc;

  if (deserializeJson(doc, pesan)) {
    Serial.println("Gagal membaca JSON!");
    return;
  }

  const char* perintah = doc["perintah"];

  // [MODIFIKASI] Mengecek apakah key "perintah" ada pada JSON
  if (perintah == nullptr) {
    Serial.println("Key 'perintah' tidak ditemukan pada JSON");
    return;
  }

  // [MODIFIKASI] Membedakan topic yang menerima pesan
  if (String(topic) == topicPerintah) {

    // Mengontrol LED
    if (String(perintah) == "ON") {
      digitalWrite(ledPin, HIGH);
      Serial.println("Perintah diterima -> LED: ON");
    } else if (String(perintah) == "OFF") {
      digitalWrite(ledPin, LOW);
      Serial.println("Perintah diterima -> LED: OFF");
    }

  } else if (String(topic) == topicBuzzer) {

    // [MODIFIKASI] Mengontrol buzzer
    if (String(perintah) == "ON") {
      digitalWrite(buzzerPin, HIGH);
      Serial.println("Perintah diterima -> Buzzer: ON");
    } else if (String(perintah) == "OFF") {
      digitalWrite(buzzerPin, LOW);
      Serial.println("Perintah diterima -> Buzzer: OFF");
    }
  }
}

// =======================
// Menghubungkan WiFi
// =======================
void hubungkanWiFi() {
  WiFi.begin(ssid, password);

  Serial.print("Menghubungkan ke WiFi");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println();
  Serial.println("WiFi berhasil terhubung!");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());
}

// =======================
// Menghubungkan MQTT
// =======================
void hubungkanMQTT() {
  while (!client.connected()) {
    Serial.print("Menghubungkan ke MQTT...");

    String clientId = "ESP8266Client-" + String(random(0xffff), HEX);

    if (client.connect(clientId.c_str())) {
      Serial.println("berhasil!");

      // Subscribe topic perintah
      client.subscribe(topicPerintah);
      Serial.print("Subscribe ke: ");
      Serial.println(topicPerintah);

      // [MODIFIKASI] Subscribe topic buzzer
      client.subscribe(topicBuzzer);
      Serial.print("Subscribe ke: ");
      Serial.println(topicBuzzer);

    } else {
      Serial.print("Gagal, rc=");
      Serial.print(client.state());
      Serial.println(" - mencoba lagi...");
      delay(2000);
    }
  }
}

// =======================
// Setup
// =======================
void setup() {
  Serial.begin(115200);

  // LED
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);

  // [MODIFIKASI] Buzzer
  pinMode(buzzerPin, OUTPUT);
  digitalWrite(buzzerPin, LOW);

  // DHT11
  dht.begin();

  // WiFi
  hubungkanWiFi();

  // MQTT
  client.setServer(mqttServer, mqttPort);
  client.setCallback(callback);
}

// =======================
// Loop
// =======================
void loop() {
  // Cek koneksi MQTT
  if (!client.connected()) {
    hubungkanMQTT();
  }

  // Memproses pesan MQTT
  client.loop();

  // =======================
  // Publish data DHT11
  // setiap 5 detik
  // =======================
  if (millis() - waktuTerakhirPublish > intervalPublish) {
    waktuTerakhirPublish = millis();

    float suhu = dht.readTemperature();

    // Cek apakah sensor berhasil dibaca
    if (!isnan(suhu)) {

      JsonDocument doc;
      doc["suhu"] = suhu;

      char buffer[128];
      serializeJson(doc, buffer);

      // Kirim data ke MQTT
      client.publish(topicData, buffer);

      Serial.print("Data terkirim: ");
      Serial.println(buffer);

    } else {
      Serial.println("Gagal membaca DHT11!");
    }
  }
}
```
<h2>Penjelasan</h2>

Program dapat ditambahkan topic baru, misal untuk mengendalikan buzzer:
```cpp
const char* topicBuzzer = "unsoed/tk245004/kelompok1/buzzer";

```
Kemudian topic tersebut juga di-subscribe setelah ESP8266 berhasil terhubung:


```cpp
client.subscribe(topicPerintah);
client.subscribe(topicBuzzer);
```

Pada fungsi callback(), topic yang menerima pesan dapat dibedakan menggunakan:
```cpp
if (String(topic) == topicPerintah) {
  // Mengendalikan LED
}
else if (String(topic) == topicBuzzer) {
  // Mengendalikan buzzer
}

```
