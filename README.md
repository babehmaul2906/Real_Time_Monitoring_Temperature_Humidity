# Real_Time_Monitoring_Temperature_Humidity
Use NodeMCU + Firebase + DHT 11
// Library yang diperlukan
#include <FirebaseESP8266.h>
#include <ESP8266WiFi.h>
#include <DHT.h>

// Mendefinisikan pin dan tipe sensor DHT
#define DHTPIN D1
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

// Harus diisi
#define FIREBASE_HOST "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
#define FIREBASE_AUTH "XXXXXXXXXXXXXXXXXXXXXXXX"
#define WIFI_SSID "XXXXXXXX"
#define WIFI_PASSWORD "XXXXXXXXX"

// mendeklarasikan objek dari FirebaseESP8266
FirebaseData firebaseData;

void setup()  {

  Serial.begin(115200);

  dht.begin();

  //Koneksi ke Wifi
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("connecting");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(500);
  }
  Serial.println();
  Serial.print("Connected with IP: ");
  Serial.println(WiFi.localIP());
  Serial.println();

  Firebase.begin(FIREBASE_HOST, FIREBASE_AUTH);
}

void loop(){

  //sensor DHT11 membaca suhu dan kelembaban
  float t = dht.readTemperature();
  float h = dht.readHumidity();

  // Memeriksa apakah sensor berhasil membaca suhu dan kelembaban
  if (isnan(t) || isnan(h)) {
    Serial.println("Gagal membaca sensor DHT11");
    return;
  }

  // Menampilkan suhu dan kelembaban pada serial monitor
  Serial.print("Suhu: ");
  Serial.print(t);
  Serial.println(" *C");
  Serial.print("Kelembaban: ");
  Serial.print(h);
  Serial.println(" %");
  Serial.println();

  // Memberikan status suhu dan kelembaban kepada firebase
  if (Firebase.setFloat(firebaseData, "/Hasil_Pembacaan/suhu", t)){
    Serial.println("Suhu terikirim");
  } else{
    Serial.println("Suhu tidak terkirim");
    Serial.println("Karena: " + firebaseData.errorReason());  
  }

  if (Firebase.setFloat(firebaseData, "/Hasil_Pembacaan/kelembaban", h)){
    Serial.println("Kelembaban terkirim");
    Serial.println();
  } else{
    Serial.println("Kelembaban tidak terkirim");
    Serial.println("Karena: " + firebaseData.errorReason());
  }

 delay(1000);
}
