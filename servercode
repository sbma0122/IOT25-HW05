#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>
#include <DHT.h>

//Default Temperature is in Celsius
#define temperatureCelsius

// BLE server name
#define bleServerName "IoT_E"

// DHT11 설정
#define DHTPIN 4
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

float temp;
float tempF;
float hum;

// Timer variables
unsigned long lastTime = 0;
unsigned long timerDelay = 30000;

bool deviceConnected = false;

// BLE UUIDs
#define SERVICE_UUID "91bad492-b950-4226-aa2b-4ede9fa42f59"

#ifdef temperatureCelsius
  BLECharacteristic bmeTemperatureCelsiusCharacteristics("cba1d466-344c-4be3-ab3f-189f80dd7518", BLECharacteristic::PROPERTY_NOTIFY);
  BLEDescriptor bmeTemperatureCelsiusDescriptor(BLEUUID((uint16_t)0x2902));
#else
  BLECharacteristic bmeTemperatureFahrenheitCharacteristics("f78ebbff-c8b7-4107-93de-889a6a06d408", BLECharacteristic::PROPERTY_NOTIFY);
  BLEDescriptor bmeTemperatureFahrenheitDescriptor(BLEUUID((uint16_t)0x2902));
#endif

BLECharacteristic bmeHumidityCharacteristics("ca73b3ba-39f6-4ab3-91ae-186dc9577d99", BLECharacteristic::PROPERTY_NOTIFY);
BLEDescriptor bmeHumidityDescriptor(BLEUUID((uint16_t)0x2903));

// BLE 서버 콜백
class MyServerCallbacks: public BLEServerCallbacks {
  void onConnect(BLEServer* pServer) {
    deviceConnected = true;
  };
  void onDisconnect(BLEServer* pServer) {
    deviceConnected = false;
  }
};

void setup() {
  Serial.begin(115200);

  // DHT11 센서 시작
  dht.begin();

  // BLE 초기화
  BLEDevice::init(bleServerName);
  BLEServer *pServer = BLEDevice::createServer();
  pServer->setCallbacks(new MyServerCallbacks());

  BLEService *bmeService = pServer->createService(SERVICE_UUID);

  // Temperature
  #ifdef temperatureCelsius
    bmeService->addCharacteristic(&bmeTemperatureCelsiusCharacteristics);
    bmeTemperatureCelsiusDescriptor.setValue("DHT temperature Celsius");
    bmeTemperatureCelsiusCharacteristics.addDescriptor(&bmeTemperatureCelsiusDescriptor);
  #else
    bmeService->addCharacteristic(&bmeTemperatureFahrenheitCharacteristics);
    bmeTemperatureFahrenheitDescriptor.setValue("DHT temperature Fahrenheit");
    bmeTemperatureFahrenheitCharacteristics.addDescriptor(&bmeTemperatureFahrenheitDescriptor);
  #endif  

  // Humidity
  bmeService->addCharacteristic(&bmeHumidityCharacteristics);
  bmeHumidityDescriptor.setValue("DHT humidity");
  bmeHumidityCharacteristics.addDescriptor(new BLE2902());

  bmeService->start();

  BLEAdvertising *pAdvertising = BLEDevice::getAdvertising();
  pAdvertising->addServiceUUID(SERVICE_UUID);
  pServer->getAdvertising()->start();
  Serial.println("Waiting a client connection to notify...");
}

void loop() {
  if (deviceConnected) {
    if ((millis() - lastTime) > timerDelay) {
      // 센서 값 읽기
      temp = dht.readTemperature();   // Celsius
      tempF = 1.8 * temp + 32;        // Fahrenheit
      hum = dht.readHumidity();

      if (isnan(temp) || isnan(hum)) {
        Serial.println("Failed to read from DHT sensor!");
        return;
      }

      #ifdef temperatureCelsius
        static char temperatureCTemp[6];
        dtostrf(temp, 6, 2, temperatureCTemp);
        bmeTemperatureCelsiusCharacteristics.setValue(temperatureCTemp);
        bmeTemperatureCelsiusCharacteristics.notify();
        Serial.print("Temperature Celsius: ");
        Serial.print(temp);
        Serial.print(" ºC");
      #else
        static char temperatureFTemp[6];
        dtostrf(tempF, 6, 2, temperatureFTemp);
        bmeTemperatureFahrenheitCharacteristics.setValue(temperatureFTemp);
        bmeTemperatureFahrenheitCharacteristics.notify();
        Serial.print("Temperature Fahrenheit: ");
        Serial.print(tempF);
        Serial.print(" ºF");
      #endif

      static char humidityTemp[6];
      dtostrf(hum, 6, 2, humidityTemp);
      bmeHumidityCharacteristics.setValue(humidityTemp);
      bmeHumidityCharacteristics.notify();
      Serial.print(" - Humidity: ");
      Serial.print(hum);
      Serial.println(" %");

      lastTime = millis();
    }
  }
}
