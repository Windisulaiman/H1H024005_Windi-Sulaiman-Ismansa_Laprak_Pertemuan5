
## Source Code
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

