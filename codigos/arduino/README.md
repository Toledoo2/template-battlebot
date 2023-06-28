# Códigos Arduino

Subir códigos **comentados** utilizados en Arduino.
#include <ESP8266WiFi.h>
#include <Servo.h>

const char ssid[] = "DukoBotsWiFi12";
const char password[] = "lachitu";
WiFiServer server(80);

int PinLED = 2;
//int estado = HIGH;

const int PWMA = 8;
const int PWMB = 7;
const int AIN2 = 6;
const int AIN1 = 5;
const int BIN1 = 2;
const int BIN2 = 1;

Servo ESC; // Crea un objeto servo para controlar el ESC (controlador electrónico de velocidad)
bool motorEncendido = false;

void setup() {
  Serial.begin(9600);
  server.begin();
  WiFi.mode(WIFI_AP);
  WiFi.softAP(ssid);

  Serial.println();
  Serial.print("Dirección IP Access Point - por defecto: ");
  Serial.println(WiFi.softAPIP());
  Serial.print("Dirección MAC Access Point: ");
  Serial.println(WiFi.softAPmacAddress());

  pinMode(PWMA, OUTPUT);
  pinMode(AIN1, OUTPUT);
  pinMode(AIN2, OUTPUT);
  pinMode(PWMB, OUTPUT);
  pinMode(BIN1, OUTPUT);
  pinMode(BIN2, OUTPUT);

  ESC.attach(9, 1000, 2000); // Conecta el ESC al pin 9
}

void loop() {
  WiFiClient client = server.available();
  if (!client) {
    return;
  }

  Serial.println("Nuevo cliente");
  while (!client.available()) {
    delay(1);
  }

  Serial.printf("Clientes conectados al Access Point: %d\n", WiFi.softAPgetStationNum());

  String peticion = client.readStringUntil('\r');

  Serial.println(peticion);
  client.flush();

  if (peticion.indexOf("/1") != -1) {
    avanzar();
  }

  if (peticion.indexOf("/4") != -1) {
    retroceder();
  }
  if (peticion.indexOf("/3") != -1) {
    derecha();
  }
  if (peticion.indexOf("/2") != -1) {
    izquierda();
  }
  if (peticion.indexOf("/on") != -1) {
    encenderMotor();
  }
  if (peticion.indexOf("/off") != -1) {
    apagarMotor();
  }
  delay(100);
  detenerMotores();

  Serial.println("Petición finalizada");
}

void avanzar() {
  digitalWrite(AIN1, LOW);
  digitalWrite(AIN2, HIGH);
  analogWrite(PWMA, 200);
  digitalWrite(BIN1, LOW);
  digitalWrite(BIN2, HIGH);
  analogWrite(PWMB, 200);
}

void retroceder() {
  digitalWrite(AIN1, HIGH);
  digitalWrite(AIN2, LOW);
  analogWrite(PWMA, 200);
  digitalWrite(BIN1, HIGH);
  digitalWrite(BIN2, LOW);
  analogWrite(PWMB, 200);
}

void derecha() {
  digitalWrite(AIN1, LOW);
  digitalWrite(AIN2, HIGH);
  analogWrite(PWMA, 150);
  digitalWrite(BIN1, LOW);
  digitalWrite(BIN2, LOW);
  analogWrite(PWMB, 150);
}

void izquierda() {
  digitalWrite(AIN1, LOW);
  digitalWrite(AIN2, LOW);
  analogWrite(PWMA, 150);
  digitalWrite(BIN1, LOW);
  digitalWrite(BIN2, HIGH);
  analogWrite(PWMB, 150);
}

void detenerMotores() {
  digitalWrite(AIN1, LOW);
  digitalWrite(AIN2, LOW);
  analogWrite(PWMA, 0);
  digitalWrite(BIN1, LOW);
  digitalWrite(BIN2, LOW);
  analogWrite(PWMB, 0);
}

void encenderMotor() {
  if (!motorEncendido) {
    ESC.write(180); // Envía una señal al ESC para encender el motor a máxima velocidad
    motorEncendido = true;
  }
}

void apagarMotor() {
  if (motorEncendido) {
    ESC.write(0); // Envía una señal al ESC para detener el motor
    motorEncendido = false;
  }
}
