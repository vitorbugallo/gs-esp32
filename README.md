# Estação de Conforto Térmico - ESP32 + DHT22 + LED

*Aluno:* Vitor Reis 
*Disciplina:* 1ESPY-2025  
*Tema:* O Futuro do Trabalho  
*Data:* 15/11/2025  

## Descrição

Sistema de monitoramento de conforto térmico para ambientes de trabalho.  
O ESP32 lê temperatura e umidade do sensor DHT22 e acende um LED verde quando a temperatura está na faixa de conforto (20-27°C).  
Os dados podem ser enviados via MQTT para integração com sistemas de supervisão.

## Circuito (Wokwi)

- *ESP32 Devkit V1*  
- *DHT22*:  
  - VCC → 3V3  
  - SDA → GPIO15  
  - GND → GND  
- *LED verde + resistor 220Ω*:  
  - Anodo (via resistor) → GPIO26  
  - Catodo → GND  

## Funcionalidades

1. *Leitura de sensores:* Temperatura e umidade via DHT22  
2. *Lógica de conforto:* LED acende entre 20-27°C  
3. *Comunicação planejada:*  
   - WiFi: Wokwi-GUEST  
   - MQTT (futuro):  
     - Broker: test.mosquitto.org:1883  
     - Tópico telemetria: gs2025/eduardo/workwell/telemetry  
     - Tópico comandos: gs2025/eduardo/workwell/cmd  
     - Payload JSON: {"temp":24.5,"umid":52.3,"conforto":true}  

## Link do Projeto Wokwi

[https://wokwi.com/projects/COLE_AQUI_O_LINK_DO_SEU_PROJETO](https://wokwi.com/projects/COLE_AQUI_O_LINK_DO_SEU_PROJETO)

## Vídeo de Demonstração

[INSERIR_LINK_DO_VÍDEO_AQUI]

## Código Principal

```cpp
#include <WiFi.h>
#include "DHTesp.h"

// CONFIG WiFi (Wokwi)
const char* WIFI_SSID = "Wokwi-GUEST";
const char* WIFI_PASS = "";

// DHT22 no GPIO15
#define PIN_DHT   15
DHTesp dht;

// LED no GPIO26
#define PIN_LED_OK 26

void setup() {
  Serial.begin(115200);
  delay(1000);
  
  pinMode(PIN_LED_OK, OUTPUT);
  digitalWrite(PIN_LED_OK, LOW);
  
  dht.setup(PIN_DHT, DHTesp::DHT22);
  
  Serial.println("Conectando ao WiFi...");
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi conectado!");
  Serial.print("IP: ");
  Serial.println(WiFi.localIP());
}

void loop() {
  delay(2000);
  
  TempAndHumidity data = dht.getTempAndHumidity();
  float temperature = data.temperature;
  float humidity    = data.humidity;
  
  if (isnan(temperature) || isnan(humidity)) {
    Serial.println("Falha ao ler DHT!");
    return;
  }
  
  Serial.print("Temp: ");
  Serial.print(temperature);
  Serial.print(" *C  | Umid: ");
  Serial.print(humidity);
  Serial.println(" %");
  
  // Conforto térmico: 20°C a 27°C
  bool conforto = (temperature >= 20 && temperature <= 27);
  
  if (conforto) {
    digitalWrite(PIN_LED_OK, HIGH);  // confortável -> LED ON
  } else {
    digitalWrite(PIN_LED_OK, LOW);   // desconforto -> LED OFF
  }
}
