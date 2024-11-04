#include <WiFi.h>
#include <ThingSpeak.h>

// Configuración de WiFi y ThingSpeak
const char* ssid = "Redmi 9";              // Reemplaza con tu SSID
const char* password = "17846547";         // Reemplaza con tu contraseña
unsigned long channelID = 2690504;         // Reemplaza con tu número de canal
const char* apiKey = "G2OFXTCYEE59ABTJ";   // Reemplaza con tu API Key

WiFiClient client;

// Pines de sensores
int pinSensorLluvia = 35;    // Pin analógico para el sensor de lluvia
int pinSensorHumedad = 34;   // Pin analógico para el sensor de humedad

// Variables para almacenamiento de datos
int valorLluviaADC = 0;
int valorHumedadADC = 0;
float voltajeHumedad = 0;
float humedad = 0;
int umbralLluvia = 300;      // Umbral para determinar si está lloviendo

void setup() {
  Serial.begin(9600); // Inicializar la comunicación serial
  WiFi.begin(ssid, password);

  // Espera a que se conecte a la red WiFi
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Conectando a WiFi...");
  }
  Serial.println("Conectado a WiFi");

  // Inicia ThingSpeak
  ThingSpeak.begin(client);
}

void loop() {
  // Lectura del sensor de lluvia
  valorLluviaADC = analogRead(pinSensorLluvia);
  String estadoLluvia = (valorLluviaADC < umbralLluvia) ? "Lluvia" : "Sin Lluvia";

  // Lectura y cálculo del porcentaje de humedad
  valorHumedadADC = analogRead(pinSensorHumedad);
  voltajeHumedad = valorHumedadADC * (3.3 / 4095.0);
  humedad = (voltajeHumedad / 3.3) * 100.0;

  // Mostrar los valores en el monitor serie
  Serial.print("Valor sensor de lluvia: ");
  Serial.println(valorLluviaADC);
  Serial.print("Estado de lluvia: ");
  Serial.println(estadoLluvia);
  Serial.print("Valor ADC de humedad: ");
  Serial.println(valorHumedadADC);
  Serial.print("Humedad del suelo: ");
  Serial.print(humedad);
  Serial.println(" %");

  // Enviar datos a ThingSpeak
  ThingSpeak.setField(1, valorLluviaADC);    // Campo 1 para el valor del sensor de lluvia
  ThingSpeak.setField(2, humedad);           // Campo 2 para el porcentaje de humedad

  // Verificar si los datos se enviaron correctamente
  int httpResponseCode = ThingSpeak.writeFields(channelID, apiKey);
  if (httpResponseCode == 200) {
    Serial.println("Datos enviados exitosamente.");
  } else {
    Serial.print("Error al enviar datos. Código de respuesta: ");
    Serial.println(httpResponseCode);
  }

  // Esperar 10 segundos antes de la siguiente lectura
  delay(10000);
}

