
# Source Code
---
````cpp
#include <Arduino_FreeRTOS.h>

void TaskBlink1(void *pvParameters);
void TaskBlink2(void *pvParameters);
void Taskprint(void *pvParameters);

void setup() {
  Serial.begin(9600);

  xTaskCreate(TaskBlink1, "task1", 128, NULL, 1, NULL);
  xTaskCreate(TaskBlink2, "task2", 128, NULL, 1, NULL);
  xTaskCreate(Taskprint,  "task3", 128, NULL, 1, NULL);

  vTaskStartScheduler();
}

void loop() {}

void TaskBlink1(void *pvParameters) {
  pinMode(9, OUTPUT);
  while (1) {
    Serial.println("Task1");
    digitalWrite(9, HIGH);
    vTaskDelay(200 / portTICK_PERIOD_MS);
    digitalWrite(9, LOW);
    vTaskDelay(200 / portTICK_PERIOD_MS);
  }
}

void TaskBlink2(void *pvParameters) {
  pinMode(8, OUTPUT);
  while (1) {
    Serial.println("Task2");
    digitalWrite(8, HIGH);
    vTaskDelay(300 / portTICK_PERIOD_MS);
    digitalWrite(8, LOW);
    vTaskDelay(300 / portTICK_PERIOD_MS);
  }
}

void Taskprint(void *pvParameters) {
  int counter = 0;
  while (1) {
    counter++;
    Serial.println(counter);
    vTaskDelay(500 / portTICK_PERIOD_MS);
  }
}
````

### Penjelasan Fungsi

| Sintaks | Penjelasan |
|---------|------------|
| `#include <Arduino_FreeRTOS.h>` | Memuat library FreeRTOS untuk Arduino. Menyediakan semua API task, scheduler, dan utilitas RTOS yang digunakan dalam program. |
| `void TaskNama(void *pvParameters)` | Deklarasi prototype fungsi task. Parameter `void *pvParameters` adalah pointer generik yang memungkinkan pengiriman data ke task saat dibuat, diisi `NULL` jika tidak ada data yang dikirim. |
| `Serial.begin(9600)` | Menginisialisasi komunikasi serial dengan baud rate 9600 bps. Dipanggil di `setup()` sebelum task dijalankan agar Serial Monitor siap digunakan oleh semua task. |
| `xTaskCreate(pvTask, pcName, usStackDepth, pvParameters, uxPriority, pxHandle)` | Membuat dan mendaftarkan task baru ke scheduler FreeRTOS. |
| `vTaskStartScheduler()` | Memulai scheduler FreeRTOS. Setelah fungsi ini dipanggil, kendali program diserahkan sepenuhnya ke scheduler dan `loop()` tidak akan pernah dieksekusi. |
| `void loop() {}` | Dikosongkan karena seluruh logika program sudah berjalan di dalam task FreeRTOS yang dikelola scheduler. |
| `pinMode(pin, OUTPUT)` | Mengatur pin sebagai output digital. Dipanggil di awal setiap task yang menggunakan LED agar pin siap digunakan sebelum `digitalWrite()` dieksekusi. |
| `while (1)` | Loop tak terbatas di dalam task. Setiap task FreeRTOS harus memiliki loop tak terbatas agar task tidak pernah return — jika task return tanpa dihapus, sistem akan crash. |
| `digitalWrite(pin, HIGH/LOW)` | Mengatur logika pin digital menjadi HIGH (5V) atau LOW (0V). Digunakan untuk menyalakan dan mematikan LED pada task blink. |
| `vTaskDelay(xTicksToDelay)` | Menunda eksekusi task selama jumlah tick yang ditentukan. Selama delay, CPU dibebaskan untuk menjalankan task lain  berbeda dengan `delay()` biasa yang memblokir seluruh CPU. |
| `portTICK_PERIOD_MS` | Konstanta konversi satuan milidetik ke tick FreeRTOS. Rumus `ms / portTICK_PERIOD_MS` digunakan agar delay dapat dinyatakan dalam milidetik yang mudah dipahami. |
| `counter++` | Increment variabel lokal task. Variabel ini bersifat lokal di dalam task `Taskprint` sehingga aman diakses tanpa mekanisme sinkronisasi tambahan. |
| `Serial.println(val)` | Mencetak nilai ke Serial Monitor dengan newline di akhir. Digunakan di beberapa task sekaligus — perlu diperhatikan bahwa akses Serial secara bersamaan dari banyak task tanpa mutex dapat menyebabkan output yang tercampur. |




# Modifikasi soal nomor 3
---
## Wiring / Koneksi Hardware
---
| Komponen | Pin Komponen | Terhubung ke | Keterangan |
|----------|-------------|--------------|------------|
| LED 1 | Anoda (+) | Arduino Pin 9 | Melalui resistor 220Ω |
| LED 1 | Katoda (-) | GND | Ground |
| LED 2 | Anoda (+) | Arduino Pin 8 | Melalui resistor 220Ω |
| LED 2 | Katoda (-) | GND | Ground |
| Potensiometer | Kaki Kiri | GND | Referensi bawah |
| Potensiometer | Kaki Tengah | Arduino A0 | Pin pembacaan ADC |
| Potensiometer | Kaki Kanan | 5V | Referensi atas |

### Dokumentasi
---
<img width="945" height="670" alt="Modifikasi soal 3 percobaan 5A" src="https://github.com/user-attachments/assets/61ce03e8-077a-4db8-a659-c7743bf052b9" />

### Kode Program
---
```cpp
#include 

void TaskBlink1(void *pvParameters);
void TaskBlink2(void *pvParameters);
void Taskprint(void *pvParameters);

volatile int potValue = 0;  // Variabel global menyimpan nilai ADC potensiometer

void setup() {
  Serial.begin(9600);

  xTaskCreate(TaskBlink1, "task1", 128, NULL, 1, NULL);
  xTaskCreate(TaskBlink2, "task2", 128, NULL, 1, NULL);
  xTaskCreate(Taskprint,  "task3", 128, NULL, 1, NULL);

  vTaskStartScheduler();
}

void loop() {}

void TaskBlink1(void *pvParameters) {
  pinMode(9, OUTPUT);
  while (1) {
    potValue  = analogRead(A0);
    int delay_ms = map(potValue, 0, 1023, 100, 1000);

    Serial.println("Task1");
    digitalWrite(9, HIGH);
    vTaskDelay(delay_ms / portTICK_PERIOD_MS);
    digitalWrite(9, LOW);
    vTaskDelay(delay_ms / portTICK_PERIOD_MS);
  }
}

void TaskBlink2(void *pvParameters) {
  pinMode(8, OUTPUT);
  while (1) {
    Serial.println("Task2");
    digitalWrite(8, HIGH);
    vTaskDelay(300 / portTICK_PERIOD_MS);
    digitalWrite(8, LOW);
    vTaskDelay(300 / portTICK_PERIOD_MS);
  }
}

void Taskprint(void *pvParameters) {
  int counter = 0;
  while (1) {
    counter++;
    Serial.print("Counter: ");
    Serial.print(counter);
    Serial.print(" | POT ADC: ");
    Serial.print(potValue);
    Serial.print(" | Delay LED1: ");
    Serial.print(map(potValue, 0, 1023, 100, 1000));
    Serial.println(" ms");
    vTaskDelay(500 / portTICK_PERIOD_MS);
  }
}
```

---

## Penjelasan Fungsi

| Sintaks | Penjelasan |
|---------|------------|
| `#include <Arduino_FreeRTOS.h>` | Memuat library FreeRTOS untuk Arduino. |
| `volatile int potValue = 0` | Variabel global bertipe `volatile` untuk menyimpan nilai ADC potensiometer. Keyword `volatile` memberitahu compiler bahwa nilai ini bisa berubah kapan saja dari task lain sehingga tidak di-cache secara lokal. |
| `xTaskCreate(pvTask, pcName, usStackDepth, pvParameters, uxPriority, pxHandle)` | Membuat dan mendaftarkan task baru ke scheduler FreeRTOS. Ketiga task dibuat dengan prioritas `1` dan stack size `128` word. |
| `vTaskStartScheduler()` | Memulai scheduler FreeRTOS. Setelah dipanggil, kendali diserahkan ke scheduler dan `loop()` tidak akan pernah dieksekusi. |
| `void loop() {}` | Dikosongkan karena seluruh logika program sudah berjalan di dalam task FreeRTOS. |
| `analogRead(A0)` | Membaca nilai tegangan analog dari pin A0 yang terhubung ke kaki tengah potensiometer. Menghasilkan nilai integer 0–1023 sesuai posisi putaran potensiometer. |
| `map(potValue, 0, 1023, 100, 1000)` | Mengkonversi nilai ADC (0–1023) menjadi rentang delay dalam milidetik (100ms–1000ms). Saat potensiometer minimum, LED berkedip cepat (100ms). Saat maksimum, LED berkedip lambat (1000ms). |
| `int delay_ms` | Variabel lokal dalam `TaskBlink1` yang menyimpan hasil konversi nilai potensiometer menjadi durasi delay. Nilainya diperbarui setiap iterasi loop sehingga kecepatan kedip LED selalu mengikuti posisi potensiometer terkini. |
| `digitalWrite(pin, HIGH/LOW)` | Mengatur logika pin digital menjadi HIGH (5V/menyala) atau LOW (0V/mati) untuk mengendalikan LED. |
| `vTaskDelay(xTicksToDelay)` | Menunda eksekusi task selama jumlah tick yang ditentukan. Selama delay, CPU dibebaskan untuk menjalankan task lain — berbeda dengan `delay()` yang memblokir seluruh CPU. |
| `portTICK_PERIOD_MS` | Konstanta konversi milidetik ke tick FreeRTOS. Rumus `ms / portTICK_PERIOD_MS` digunakan agar delay dapat dinyatakan dalam milidetik. |
| `while (1)` | Loop tak terbatas di dalam setiap task. Wajib ada agar task tidak pernah return — jika task return tanpa dihapus eksplisit, sistem akan crash. |
| `Serial.print()` / `Serial.println()` | Mencetak data ke Serial Monitor. Pada `Taskprint`, digunakan untuk menampilkan nilai counter, nilai ADC potensiometer, dan delay aktif LED1 dalam satu baris per 500ms. |
| `counter++` | Increment variabel lokal `counter` di `Taskprint`. Variabel ini bersifat lokal sehingga aman diakses tanpa mekanisme sinkronisasi tambahan. |

---

