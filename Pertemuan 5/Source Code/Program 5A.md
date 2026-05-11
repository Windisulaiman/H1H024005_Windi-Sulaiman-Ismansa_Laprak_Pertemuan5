
## Source Code
````cpp
#include <Arduino_FreeRTOS.h>

#define PIN_LED1  8
#define PIN_LED2  7

#define DELAY_TASK1   200
#define DELAY_TASK2   300
#define DELAY_PRINT   500

void TaskBlink1(void *pvParameters);
void TaskBlink2(void *pvParameters);
void TaskPrint(void *pvParameters);

void setup() {
  Serial.begin(9600);

  xTaskCreate(TaskBlink1, "task1", 128, NULL, 1, NULL);
  xTaskCreate(TaskBlink2, "task2", 128, NULL, 1, NULL);
  xTaskCreate(TaskPrint,  "task3", 128, NULL, 1, NULL);

  vTaskStartScheduler();
}

void loop() {
}

void TaskBlink1(void *pvParameters) {
  pinMode(PIN_LED1, OUTPUT);

  while (1) {
    Serial.println("Task1");
    digitalWrite(PIN_LED1, HIGH);
    vTaskDelay(DELAY_TASK1 / portTICK_PERIOD_MS);
    digitalWrite(PIN_LED1, LOW);
    vTaskDelay(DELAY_TASK1 / portTICK_PERIOD_MS);
  }
}

void TaskBlink2(void *pvParameters) {
  pinMode(PIN_LED2, OUTPUT);

  while (1) {
    Serial.println("Task2");
    digitalWrite(PIN_LED2, HIGH);
    vTaskDelay(DELAY_TASK2 / portTICK_PERIOD_MS);
    digitalWrite(PIN_LED2, LOW);
    vTaskDelay(DELAY_TASK2 / portTICK_PERIOD_MS);
  }
}

void TaskPrint(void *pvParameters) {
  int counter = 0;

  while (1) {
    Serial.println(++counter);
    vTaskDelay(DELAY_PRINT / portTICK_PERIOD_MS);
  }
}
````

