#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// Definindo as portas
#define RELAY_PIN 8           // Porta de controle do relé
#define MOISTURE_SENSOR_PIN A0 // Porta analógica do sensor de umidade

// Definindo o OLED
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 32
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

// Definindo os limites do sensor de umidade (ajuste conforme necessário)
const int DRY_VALUE = 800;    // Valor analógico quando o solo está seco
const int WET_VALUE = 300;    // Valor analógico quando o solo está molhado

void setup() {
  // Configuração do Serial para debugging
  Serial.begin(9600);

  // Configurando as portas
  pinMode(RELAY_PIN, OUTPUT);

  // Configurando o display OLED
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) { 
    Serial.println(F("Falha ao iniciar o display OLED"));
    for (;;);
  }
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);

  // Inicializando o relé como desligado
  digitalWrite(RELAY_PIN, LOW);
}

void loop() {
  // Ler o valor analógico do sensor de umidade
  int sensorValue = analogRead(MOISTURE_SENSOR_PIN);

  // Mapear o valor analógico para porcentagem (0 = seco, 100 = molhado)
  int moisturePercentage = map(sensorValue, DRY_VALUE, WET_VALUE, 0, 100);
  moisturePercentage = constrain(moisturePercentage, 0, 100); // Garantir valores entre 0 e 100

  // Exibir o valor no display OLED
  display.clearDisplay();
  display.setCursor(0, 0);
  display.print("Umidade do Solo: ");
  display.print(moisturePercentage);
  display.print("%");
  display.display();

  // Controlar o relé/motor conforme o nível de umidade
  if (moisturePercentage < 30) { // Umidade abaixo de 30%
    digitalWrite(RELAY_PIN, HIGH); // Liga o motor
    Serial.println("Motor ligado para irrigacao");
  } else {
    digitalWrite(RELAY_PIN, LOW); // Desliga o motor
    Serial.println("Motor desligado");
  }

  // Exibir o valor no monitor serial
  Serial.print("Valor do sensor: ");
  Serial.print(sensorValue);
  Serial.print(" - Umidade: ");
  Serial.print(moisturePercentage);
  Serial.println("%");

  // Atraso para estabilização
  delay(1000);
}
