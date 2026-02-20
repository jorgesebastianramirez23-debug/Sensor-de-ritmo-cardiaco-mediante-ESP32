# Sensor-de-ritmo-cardiaco-mediante-ESP32
Proyecto Sensor de Ritmo Cardiaco 

#include "BluetoothSerial.h"

BluetoothSerial SerialBT;

#define PULSE_PIN 34
#define MUESTRAS 10

int buffer[MUESTRAS];
int indice = 0;

unsigned long ultimoLatido = 0;
float BPM = 0;
bool detectandoPulso = false;

int leerFiltrado() {
  buffer[indice] = analogRead(PULSE_PIN);
  indice = (indice + 1) % MUESTRAS;

  long suma = 0;
  for (int i = 0; i < MUESTRAS; i++) {
    suma += buffer[i];
  }

  return suma / MUESTRAS;
}

void setup() {
  Serial.begin(115200);
  SerialBT.begin("Monitor_Cardiaco");

  analogReadResolution(12);
  analogSetPinAttenuation(PULSE_PIN, ADC_11db);
}

void loop() {

  int lectura = leerFiltrado();

  Serial.println(lectura);  // gráfico

  if (lectura > 2000 && !detectandoPulso) {

    detectandoPulso = true;

    unsigned long ahora = millis();
    unsigned long intervalo = ahora - ultimoLatido;

    if (intervalo > 400 && intervalo < 1500) {
      BPM = 60000.0 / intervalo;
    }

    ultimoLatido = ahora;
  }

  if (lectura < 1800) {
    detectandoPulso = false;
  }

  // 🔥 SOLO CAMBIO AQUÍ → 5000 ms = 5 segundos
  static unsigned long tiempoEnvio = 0;

  if (millis() - tiempoEnvio > 5000) {

    SerialBT.print("BPM: ");
    
    if (BPM > 0)
      SerialBT.println(BPM);
    else
      SerialBT.println("----");

    tiempoEnvio = millis();
  }

  delay(10);
}
