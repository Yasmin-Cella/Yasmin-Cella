#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

//alunos: Jheniffer, Viccenzo e Yasmin - 2024

// Definindo as portas
#define RELAY_PIN 8           // Porta de controle do relé
#define MOISTURE_SENSOR_PIN A0 // Porta analógica do sensor de umidade

// Definindo o OLED
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

// Definindo os limites do sensor de umidade
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
  moisturePercentage = constrain(moisturePercentage, 0, 100);

  // Exibir o valor no display OLED
  display.clearDisplay();
  display.setCursor(0, 0);
  display.print("Umidade do Solo: ");
  display.print(moisturePercentage);
  display.print("%");
  display.display();

  // Controlar o relé/motor conforme o nível de umidade
  if (moisturePercentage < 30) { // Umidade abaixo de 30%
    display.clearDisplay();
    desenharSol();
    display.display();
    delay(5000);
    display.clearDisplay();
    display.print("Solo seco!");
    display.display();
    delay(5000);

    digitalWrite(RELAY_PIN, HIGH); // Liga o motor

    display.clearDisplay();
    desenharNuvem();
    display.display();
    delay(5000);
    display.clearDisplay();
    display.print("Motor ligado.");
    display.display();
    delay(5000);
  } else {
    digitalWrite(RELAY_PIN, LOW); // Desliga o motor

    display.clearDisplay();
    desenharFlor();
    display.display();
    delay(5000);
    display.clearDisplay();
    display.print("Umidade ideal.");
    display.display();
    delay(5000);
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

void desenharSol() {
  int centerX = 64; // Centro do sol no eixo X
  int centerY = 32; // Centro do sol no eixo Y
  int radius = 10;  // Raio do círculo central

  // Desenhar o círculo central (o corpo do sol)
  display.fillCircle(centerX, centerY, radius, SSD1306_WHITE);

  // Desenhar os raios ao redor do sol
  for (int angle = 0; angle < 360; angle += 30) {
    // Calcular as posições finais dos raios usando seno e cosseno
    int xStart = centerX + radius * cos(radians(angle));
    int yStart = centerY + radius * sin(radians(angle));
    int xEnd = centerX + (radius + 10) * cos(radians(angle));
    int yEnd = centerY + (radius + 10) * sin(radians(angle));

    // Desenhar a linha do raio
    display.drawLine(xStart, yStart, xEnd, yEnd, SSD1306_WHITE);
  }
}

void desenharNuvem() {
  // Coordenadas e dimensões da nuvem
  int xCentro = 64; // Centro da nuvem no eixo X
  int yCentro = 20; // Centro da nuvem no eixo Y
  int larguraNuvem = 50; // Largura da nuvem
  int alturaNuvem = 20; // Altura da nuvem

  // Desenhar a parte superior da nuvem com curvas
  display.drawCircle(xCentro - 20, yCentro, 10, SSD1306_WHITE); // Bolha esquerda
  display.drawCircle(xCentro, yCentro, 15, SSD1306_WHITE);      // Bolha central
  display.drawCircle(xCentro + 20, yCentro, 10, SSD1306_WHITE); // Bolha direita

  // Desenhar a base da nuvem como uma linha reta
  display.drawLine(xCentro - 30, yCentro + 10, xCentro + 30, yCentro + 10, SSD1306_WHITE);

  // Desenhar a chuva (linhas verticais abaixo da nuvem)
  for (int i = -20; i <= 20; i += 10) { // Linhas espaçadas horizontalmente
    display.drawLine(xCentro + i, yCentro + 15, xCentro + i, yCentro + 25, SSD1306_WHITE);
  }
}

void desenharFlor() {
  int xCentro = 64; // Centro da flor no eixo X
  int yCentro = 20; // Centro da flor no eixo Y
  int raioPetala = 8; // Raio das pétalas
  int raioMiolo = 5; // Raio do miolo

  // Desenhar as pétalas
  for (int angle = 0; angle < 360; angle += 45) { // Pétalas espaçadas a cada 45 graus
    int xPetala = xCentro + raioPetala * 2 * cos(radians(angle));
    int yPetala = yCentro + raioPetala * 2 * sin(radians(angle));
    display.drawCircle(xPetala, yPetala, raioPetala, SSD1306_WHITE);
  }

  // Desenhar o miolo da flor
  display.fillCircle(xCentro, yCentro, raioMiolo, SSD1306_WHITE);

  // Desenhar o caule
  display.drawLine(xCentro, yCentro + raioPetala * 2, xCentro, yCentro + 40, SSD1306_WHITE);

  // Desenhar as folhas
  display.drawTriangle(xCentro, yCentro + 30, xCentro - 10, yCentro + 25, xCentro - 10, yCentro + 35, SSD1306_WHITE); // Folha esquerda
  display.drawTriangle(xCentro, yCentro + 30, xCentro + 10, yCentro + 25, xCentro + 10, yCentro + 35, SSD1306_WHITE); // Folha direita
}
