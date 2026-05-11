# Source Code
----
````cpp
#include <Arduino_FreeRTOS.h>
#include <queue.h>

struct readings {
  int temp;
  int h;
};

QueueHandle_t my_queue;

void setup() {
  Serial.begin(9600);
  my_queue = xQueueCreate(1, sizeof(struct readings));
  xTaskCreate(read_data, "read sensors", 128, NULL, 0, NULL);
  xTaskCreate(display, "display", 128, NULL, 0, NULL);
}

void loop() {}

void read_data(void *pvParameters) {
  struct readings x;
  for (;;) {
    x.temp = 54;
    x.h = 30;
    xQueueSend(my_queue, &x, portMAX_DELAY);
    vTaskDelay(100);
  }
}

void display(void *pvParameters) {
  struct readings x;
  for (;;) {
    if (xQueueReceive(my_queue, &x, portMAX_DELAY) == pdPASS) {
      Serial.print("temp = ");
      Serial.println(x.temp);
      Serial.print("humidity = ");
      Serial.println(x.h);
    }
  }
}
````
### Penjelasan Fungsi

| Sintaks | Penjelasan |
|---------|------------|
| `#include <Arduino_FreeRTOS.h>` | Memuat library FreeRTOS untuk Arduino yang menyediakan API task, scheduler, dan mekanisme sinkronisasi antar task. |
| `#include <queue.h>` | Menambahkan header khusus queue FreeRTOS yang menyediakan API untuk membuat dan mengoperasikan queue sebagai media komunikasi antar task. |
| `struct readings { int temp; int h; }` | Mendefinisikan struktur data kustom bernama `readings` yang menampung dua field: `temp` untuk suhu dan `h` untuk kelembaban. Digunakan sebagai satuan data yang dikirim melalui queue. |
| `QueueHandle_t my_queue` | Deklarasi handle queue bertipe `QueueHandle_t`. Handle ini digunakan sebagai referensi untuk semua operasi pada queue seperti send dan receive. |
| `Serial.begin(9600)` | Menginisialisasi komunikasi serial dengan baud rate 9600 bps agar output dapat ditampilkan di Serial Monitor. |
| `xQueueCreate(uxQueueLength, uxItemSize)` | Membuat queue baru di heap FreeRTOS. Parameter pertama adalah kapasitas maksimal item dalam queue (1 item), parameter kedua adalah ukuran tiap item dalam byte menggunakan `sizeof(struct readings)`. |
| `xTaskCreate(pvTask, pcName, usStackDepth, pvParameters, uxPriority, pxHandle)` | Membuat dan mendaftarkan task baru ke scheduler FreeRTOS. Kedua task dibuat dengan prioritas `0` (terendah) agar scheduler membagi waktu CPU secara merata. |
| `struct readings x` | Deklarasi variabel lokal bertipe `struct readings` di dalam masing-masing task. Variabel ini digunakan sebagai buffer sementara untuk menyimpan data sebelum dikirim atau setelah diterima dari queue. |
| `x.temp = 54` | Mengisi field `temp` pada struct dengan nilai simulasi suhu 54°C. Pada implementasi nyata, baris ini diganti dengan pembacaan sensor sesungguhnya. |
| `x.h = 30` | Mengisi field `h` pada struct dengan nilai simulasi kelembaban 30%. Sama seperti `x.temp`, pada implementasi nyata diganti dengan pembacaan sensor. |
| `xQueueSend(xQueue, pvItemToQueue, xTicksToWait)` | Mengirim satu item ke dalam queue. Parameter pertama adalah handle queue, kedua adalah pointer ke data yang dikirim, ketiga adalah waktu tunggu maksimal jika queue penuh. `portMAX_DELAY` berarti task akan menunggu tanpa batas hingga queue tersedia. |
| `vTaskDelay(xTicksToDelay)` | Menunda eksekusi task selama jumlah tick yang ditentukan (100 tick). Selama delay, CPU dibebaskan untuk menjalankan task lain, dalam hal ini task `display`. |
| `xQueueReceive(xQueue, pvBuffer, xTicksToWait)` | Mengambil satu item dari queue dan menyimpannya ke buffer yang ditunjuk. Task akan diblokir selama `portMAX_DELAY` jika queue masih kosong, sehingga tidak membuang siklus CPU secara sia-sia. |
| `pdPASS` | Konstanta FreeRTOS bernilai `1` yang menandakan operasi queue berhasil. Digunakan sebagai kondisi pengecekan apakah `xQueueReceive()` berhasil mengambil data sebelum data ditampilkan. |
| `Serial.print(val)` | Mencetak data ke Serial Monitor tanpa newline. Digunakan untuk mencetak label teks sebelum nilai ditampilkan pada baris yang sama. |
| `Serial.println(val)` | Mencetak data ke Serial Monitor dengan newline di akhir baris. Digunakan untuk mencetak nilai `temp` dan `h` agar setiap data tampil pada baris terpisah. |

# Modifikasi Soal nomor 3
---

### Wiring Rangkaian

| Pin Sensor DHT22 | Terhubung ke | Keterangan |
|------------------|--------------|------------|
| VCC | Arduino 5V | Sumber tegangan sensor |
| GND | Arduino GND | Ground sensor |
| DATA (SDA) | Arduino Pin 2 | Jalur data digital sensor ke mikrokontroler |

---
### Dokumentasi
---
[Modifikasi soal nomor 3](https://github.com/user-attachments/assets/74e78121-68fd-41fc-96ee-08c6cbbd01e3)

# 📖 Referensi Fungsi — DHT Sensor + Simulasi Task (millis-based)

---

### Kode Program

```cpp
#include <DHT.h>

#define DHTPIN   2
#define DHTTYPE  DHT22

DHT dht(DHTPIN, DHTTYPE);

struct readings {
  float temp;
  float h;
  bool  valid;
};

readings queueSlot;
bool     newData = false;

unsigned long lastReadTime = 0;
const unsigned long READ_INTERVAL = 2000;

void setup() {
  Serial.begin(9600);
  dht.begin();
  Serial.println(F("=== DHT Monitor dimulai ==="));
  Serial.println(F("Menunggu pembacaan pertama..."));
  Serial.println(F("--------------------"));
}

void task_read_data() {
  unsigned long now = millis();
  if (now - lastReadTime < READ_INTERVAL) return;
  lastReadTime = now;

  readings data;
  data.h    = dht.readHumidity();
  data.temp = dht.readTemperature();
  data.valid = !(isnan(data.h) || isnan(data.temp));

  queueSlot = data;
  newData   = true;
}

void task_display() {
  if (!newData) return;
  newData = false;

  readings data = queueSlot;

  if (data.valid) {
    Serial.print(F("Suhu      : "));
    Serial.print(data.temp, 1);
    Serial.println(F(" °C"));
    Serial.print(F("Kelembaban: "));
    Serial.print(data.h, 1);
    Serial.println(F(" %"));
    float hi = dht.computeHeatIndex(data.temp, data.h, false);
    Serial.print(F("Heat Index: "));
    Serial.print(hi, 1);
    Serial.println(F(" °C"));
    Serial.println(F("--------------------"));
  } else {
    Serial.println(F("ERROR: Gagal membaca sensor DHT!"));
    Serial.println(F("Periksa kabel dan koneksi sensor."));
    Serial.println(F("--------------------"));
  }
}

void loop() {
  task_read_data();
  task_display();
}
```

### Penjelasan Fungsi

| Sintaks | Penjelasan |
|---------|------------|
| `#include <DHT.h>` | Memuat library DHT dari Adafruit untuk membaca sensor suhu dan kelembaban seri DHT11/DHT22. |
| `#define DHTPIN 2` | Mendefinisikan konstanta pin data sensor DHT yang terhubung ke pin digital 2 Arduino. |
| `#define DHTTYPE DHT22` | Mendefinisikan tipe sensor yang digunakan. DHT22 memiliki akurasi lebih tinggi dibanding DHT11. |
| `DHT dht(DHTPIN, DHTTYPE)` | Membuat objek `dht` dari class DHT dengan konfigurasi pin dan tipe sensor yang sudah didefinisikan. |
| `struct readings { float temp; float h; bool valid; }` | Mendefinisikan struktur data kustom untuk menampung hasil pembacaan sensor: suhu, kelembaban, dan flag validitas data. |
| `readings queueSlot` | Deklarasi variabel global bertipe `struct readings` yang berfungsi sebagai slot penyimpanan data, menggantikan peran queue FreeRTOS berukuran 1. |
| `bool newData = false` | Flag penanda ketersediaan data baru. Bernilai `true` setelah sensor berhasil dibaca, dan kembali `false` setelah data ditampilkan — menggantikan mekanisme `xQueueSend` dan `xQueueReceive`. |
| `unsigned long lastReadTime = 0` | Menyimpan timestamp terakhir kali sensor dibaca. Digunakan bersama `millis()` untuk mengukur selang waktu antar pembacaan. |
| `const unsigned long READ_INTERVAL = 2000` | Konstanta interval pembacaan sensor dalam milidetik (2000ms = 2 detik), setara dengan `vTaskDelay(2000)` pada versi FreeRTOS. |
| `Serial.begin(9600)` | Menginisialisasi komunikasi serial dengan baud rate 9600 bps agar output dapat ditampilkan di Serial Monitor. |
| `dht.begin()` | Menginisialisasi sensor DHT dan menyiapkan pin data untuk komunikasi. Wajib dipanggil di `setup()` sebelum fungsi baca sensor digunakan. |
| `Serial.println(F("..."))` | Mencetak string ke Serial Monitor dengan newline di akhir. Makro `F()` menyimpan string langsung di Flash memory (program memory) sehingga tidak menghabiskan RAM yang terbatas pada Arduino. |
| `millis()` | Mengembalikan jumlah milidetik sejak Arduino pertama kali dinyalakan bertipe `unsigned long`. Digunakan untuk mengukur selang waktu tanpa memblokir eksekusi program seperti `delay()`. |
| `now - lastReadTime < READ_INTERVAL` | Pengecekan apakah interval 2 detik belum tercapai. Jika belum, fungsi langsung `return` tanpa melakukan pembacaan — teknik ini disebut non-blocking timing. |
| `dht.readHumidity()` | Membaca nilai kelembaban relatif dari sensor DHT22 dalam satuan persen (%). Mengembalikan `NaN` jika pembacaan gagal. |
| `dht.readTemperature()` | Membaca nilai suhu dari sensor DHT22 dalam satuan Celsius. Mengembalikan `NaN` jika pembacaan gagal. Parameter `true` dapat ditambahkan untuk menghasilkan nilai dalam Fahrenheit. |
| `isnan(val)` | Mengecek apakah nilai berupa `NaN` (Not a Number). Digunakan untuk memvalidasi hasil pembacaan sensor — jika sensor tidak terbaca, hasilnya adalah `NaN`. |
| `data.valid = !(isnan(data.h) \|\| isnan(data.temp))` | Menetapkan flag validitas data. Bernilai `true` hanya jika kedua nilai suhu dan kelembaban bukan `NaN`, artinya pembacaan sensor berhasil. |
| `queueSlot = data` | Menyalin seluruh isi struct `data` ke `queueSlot`. Operasi ini menimpa data lama tanpa pengecekan apakah sudah dibaca, setara dengan `xQueueOverwrite()` pada FreeRTOS. |
| `newData = true` | Menandai bahwa ada data baru yang siap dikonsumsi oleh `task_display()`, setara dengan sinyal yang dikirim setelah `xQueueSend()` berhasil. |
| `if (!newData) return` | Jika tidak ada data baru, `task_display()` langsung keluar tanpa melakukan apapun — mencegah pemrosesan data lama yang sudah pernah ditampilkan. |
| `newData = false` | Mengonsumsi flag data baru, menandai bahwa data sudah diambil dan diproses, setara dengan data yang sudah berhasil di-receive dari queue FreeRTOS. |
| `dht.computeHeatIndex(temp, humidity, isFahrenheit)` | Menghitung Heat Index (indeks panas) berdasarkan kombinasi suhu dan kelembaban. Parameter ketiga `false` menandakan input dalam Celsius. Heat Index mencerminkan suhu yang dirasakan tubuh manusia. |
| `Serial.print(data.temp, 1)` | Mencetak nilai float ke Serial Monitor dengan 1 angka di belakang koma. Parameter kedua menentukan jumlah digit desimal yang ditampilkan. |
| `task_read_data()` | Fungsi simulasi Task 1 yang dipanggil dari `loop()`. Bertugas membaca sensor setiap 2 detik menggunakan pendekatan non-blocking berbasis `millis()`. |
| `task_display()` | Fungsi simulasi Task 2 yang dipanggil dari `loop()`. Bertugas menampilkan data ke Serial Monitor hanya ketika ada data baru yang tersedia. |

---
