## Source Code
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
