## NOMES
## Gabriel Henrique Padula
## Rodrigo Hydeik Mendes Nakata
## LINK
https://wokwi.com/projects/400005643722175489
# Sistema de Monitoramento de Qualidade da Água

## Descrição

Este projeto é um protótipo de monitoramento de água utilizando um microcontrolador Arduino. O sistema coleta informações como pH, salinidade, oxigênio e temperatura da água, exibindo-os em um LCD 20x4,quando os dados são irregulares o led vermelho é ligado e quando esta tudo normal o verde fica ligado. Os dados também são enviados para a porta serial para visualização e registro.

Como os simuladores não possuem alguns dos sensores necessários, utilizamos potenciômetros para simular os valores.

## Funcionalidades

- Medição da temperatura da água usando um sensor DS18B20
- Medição do pH da água
- Medição da salinidade da água
- Medição do oxigênio dissolvido na água
- Exibição dos dados em um display LCD I2C
- -Quando os dados estão acima ou abaixo do esperado o led vermelho ira ligar
- Envio dos dados para o monitor serial para visualização e registro

## Instruções de Uso

### Materiais Necessários

- Microcontrolador Arduino (ex: Arduino Uno)
- Potenciômetro para simular o pH
- Potenciômetro para simular o oxigênio dissolvido
- Potenciômetro para simular o sensor de salinidade
- Sensor de temperatura DS18B20
- Display LCD com interface I2C (20x4)
- ir-receiver e ir-remote(sensor de infravermelho e controle)
- Protoboard e jumpers

### Conexões

1. **Sensor de Temperatura DS18B20**
   - VCC -> VCC da Protoboard
   - GND -> GND da Protoboard
   - Data -> Pino digital 13 do Arduino (caso troque o pino, atualize a variável `temp` no código)

2. **Display LCD com I2C**
   - VCC -> VCC da Protoboard
   - GND -> GND da Protoboard
   - SDA -> SDA do Arduino (A4 no Arduino Uno)
   - SCL -> SCL do Arduino (A5 no Arduino Uno)

3. **Potenciômetro (Simulador de pH)**
   - Sinal -> Pino analógico A0 do Arduino (caso troque o pino, atualize a variável `phPin` no código)

4. **Potenciômetro (Simulador de Salinidade)**
   - Sinal -> Pino analógico A1 do Arduino (caso troque o pino, atualize a variável `salinidadePin` no código)

5. **Potenciômetro (Simulador de Oxigênio Dissolvido)**
   - Sinal -> Pino analógico A2 do Arduino (caso troque o pino, atualize a variável `oxigenioPin` no código)

6. **LEDs para Indicação de Limite**
   - LED Vermelho (Pino 7): Indica valores acima do limite.
   - LED Verde (Pino 6): Indica valores dentro do limite.

### Requisitos

- Arduino IDE
- Arduino compatível
- Sensores conectados corretamente

### Dependências

- Biblioteca OneWire (utilizada para interagir com o sensor de temperatura DS18B20)
- Biblioteca LiquidCrystal_I2C (utilizada para controlar o display LCD via interface I2C)
- Biblioteca DallasTemperature (utilizada para interagir com o sensor de temperatura DS18B20)
- Biblioteca IRremote (para interação com o infravermelho)

## Instalação

### Instalação das Bibliotecas

1. Abra a Arduino IDE.
2. Vá em Sketch > Include Library > Manage Libraries.
3. No Gerenciador de Bibliotecas, procure por cada uma das bibliotecas e instale-as.

### Carregamento do Código

1. Conecte o Arduino ao computador via USB.
2. Abra a Arduino IDE.
3. Copie e cole o código fornecido.
4. Selecione a placa Arduino correta em Tools > Board.
5. Selecione a porta serial correta em Tools > Port.
6. Carregue o sketch para o Arduino clicando no botão de upload (seta para a direita).

### Código
```cpp
#include <OneWire.h>
#include <DallasTemperature.h>
#include <LiquidCrystal_I2C.h>
#include <IRremote.h>

#define iv_receiver 12
#define temp 13
#define phPin A0
#define salinidadePin A1
#define oxigenioPin A2
#define irReceiverPin  11

#define red 7
#define green 6

#define TIPO_TEMPERATURA 1
#define TIPO_PH 2
#define TIPO_SALINIDADE 3
#define TIPO_OXIGENIO 4
#define TIPO_NONE 0



OneWire oneWire(temp);
DallasTemperature sensors(&oneWire);
IRrecv receiver(irReceiverPin);
LiquidCrystal_I2C lcd(0x27, 20, 4);

byte grauSimbolo[8] = {
  0b00111,
  0b00101,
  0b00111,
  0b00000,
  0b00000,
  0b00000,
  0b00000,
  0b00000
};

int tipo = TIPO_NONE;


void inicio(){
  lcd.setCursor(0,0);
  lcd.print("Aperte 1-Temperatura");
  lcd.setCursor(0,1);
  lcd.print("2-pH");
  lcd.setCursor(0,2);
  lcd.print("3-Salinidade");
  lcd.setCursor(0,3);
  lcd.print("4-Oxigenio");
  }


void setup() {
  pinMode(red, OUTPUT);
  pinMode(green, OUTPUT);
  Serial.begin(9600);
  Serial.println("Iniciando Teste: ");
  lcd.init();
  lcd.backlight();
  lcd.createChar(0, grauSimbolo);
  lcd.setCursor(0,0);
  inicio();
  sensors.begin();
  receiver.enableIRIn();
  pinMode(12, OUTPUT);
}

void barradeProgresso(int valor, int maximo) {
  lcd.setCursor(0,1);
  int tamanho = 16;
  int progresso = map(valor, 0, maximo, 0, tamanho);
  lcd.print("[");
  for (int i = 0; i < tamanho; ++i) {
    if (i < progresso) {
      lcd.print("#");
    } else {
      lcd.print(" ");
    }
  }
  lcd.print("]");
}

void led_limite(float maximo,float minimo,float valor){
  if(maximo < valor){
    digitalWrite(red, HIGH);
    digitalWrite(green, LOW);

  }
  else if(valor < minimo){
    digitalWrite(red, HIGH);
    digitalWrite(green, LOW);

  }
  else{
    digitalWrite(green, HIGH);
    digitalWrite(red, LOW);

  }
}


void loop() { 

  if (receiver.decode()) {
    int command = receiver.decodedIRData.command;
    //Serial.println(command);
    
    switch (command) {
      case 48: 
        tipo = TIPO_TEMPERATURA;
        break;
      case 24: 
        tipo = TIPO_PH;
        break;
      case 122: 
        tipo = TIPO_SALINIDADE;
        break;
      case 16: 
        tipo = TIPO_OXIGENIO;
        break;
      default:
        tipo = TIPO_NONE;
        break;
    }
    
    receiver.resume();
  }

  switch (tipo) {
    case TIPO_TEMPERATURA:
      sensors.requestTemperatures();
      lcd.clear();
      lcd.setCursor(0,0);
      lcd.print("Temp:");
      lcd.print(sensors.getTempCByIndex(0));
      lcd.write(byte(0)); 
      lcd.print("C ");      
      Serial.print("Temperatura: ");
      Serial.println(sensors.getTempCByIndex(0));
      barradeProgresso(sensors.getTempCByIndex(0), 40);
      led_limite(30,26,sensors.getTempCByIndex(0));
      delay(1500);   
      break;

    case TIPO_PH:
      {
        int a0Value = analogRead(phPin);
        float pH = (float)a0Value * 14.0 / 1023.0;
        lcd.clear();
        lcd.setCursor(0,0);
        lcd.print("pH:");
        lcd.print(pH);      
        Serial.print("pH: ");
        Serial.println(pH); 
        barradeProgresso(pH, 14);
        led_limite(8.4,7.8,pH);
        delay(1500);   
      }
      break;

    case TIPO_SALINIDADE:
      {
        int a1Value = analogRead(salinidadePin);
        int salinidade = map(a1Value, 0, 1023, 0, 40);
        lcd.clear();
        lcd.setCursor(0,0);
        lcd.print("Salinidade:");
        lcd.print(salinidade);
        lcd.print("% ");
        Serial.print("Salinidade: ");
        Serial.print(salinidade);
        Serial.println("%");
        barradeProgresso(salinidade, 40);
        led_limite(38,33,salinidade);
        delay(1500);   
      }
      break;

    case TIPO_OXIGENIO:
      {
        int a2Value = analogRead(oxigenioPin);
        float oxigenio = map(a2Value, 0, 1023, 0, 200) / 10.0;
        lcd.clear();
        lcd.setCursor(0,0);
        lcd.print("Oxigenio:");
        lcd.print(oxigenio);
        lcd.print("mg/L ");
        Serial.print("Oxigenio: ");
        Serial.print(oxigenio);
        Serial.println("mg/L");
        barradeProgresso(oxigenio, 20); 
        led_limite(8,5,oxigenio);
        delay(1500);   
      }
      break;

    case TIPO_NONE:
    default:      
      break;
  }
}
```
