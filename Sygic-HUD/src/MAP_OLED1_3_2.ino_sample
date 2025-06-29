#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SH110X.h>

// Định nghĩa cho OLED 1.3 inch SH110x (128x64)
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET    -1 // Reset pin không dùng
#define OLED_I2C_ADDRESS 0x3C
Adafruit_SH1106G display = Adafruit_SH1106G(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

// Định nghĩa UUID từ bảng của Sygic
#define SERVICE_UUID        "DD3F0AD1-6239-4E1F-81F1-91F6C9F01D86"
#define CHAR_INDICATE_UUID  "DD3F0AD2-6239-4E1F-81F1-91F6C9F01D86"
#define CHAR_WRITE_UUID     "DD3F0AD3-6239-4E1F-81F1-91F6C9F01D86"

BLECharacteristic *pWriteCharacteristic;
bool deviceConnected = false;
uint32_t initialDistance = 0;
uint32_t currentDistance = 0;

// Hàm vẽ mũi tên rẽ trái
void drawLeftArrow() {
  display.fillRect(58, 5, 11, 25, SH110X_WHITE); // Khối đường thẳng đứng
  display.fillRect(45, 5, 13, 10, SH110X_WHITE); // Thân mũi tên
  display.fillTriangle(32, 10, 45, 0, 45, 20, SH110X_WHITE); // Đầu mũi tên
}

// Hàm vẽ mũi tên rẽ phải
void drawRightArrow() {
  display.fillRect(58, 5, 11, 25, SH110X_WHITE); // Khối đường thẳng đứng
  display.fillRect(69, 5, 13, 10, SH110X_WHITE); // Thân mũi tên
  display.fillTriangle(95, 10, 82, 0, 82, 20, SH110X_WHITE); // Đầu mũi tên
}

// Hàm vẽ mũi tên đi thẳng
void drawStraightArrow() {
  display.fillRect(58, 10, 11, 19, SH110X_WHITE);
  display.fillTriangle(63, 2, 47, 14, 79, 14, SH110X_WHITE);
}

// Hàm vẽ icon chữ "H" bằng khối hình học
void drawDestinationIcon() {
  display.fillRect(50, 10, 8, 30, SH110X_WHITE); // Thanh dọc trái
  display.fillRect(70, 10, 8, 30, SH110X_WHITE); // Thanh dọc phải
  display.fillRect(58, 22, 12, 6, SH110X_WHITE); // Thanh ngang ở giữa
}

// Hàm vẽ thanh tỷ lệ khoảng cách - Thanh sát dưới, chữ ở trên thanh
void drawDistanceBar(uint32_t distanceTravelled, uint32_t maxDistance) {
  const int barX = 10;
  const int barY = SCREEN_HEIGHT - 12; // Thanh sát mép dưới (64 - 12 = 52)
  const int barWidth = 108;
  const int barHeight = 12;

  display.drawRect(barX, barY, barWidth, barHeight, SH110X_WHITE);
  int fillWidth = map(distanceTravelled, 0, maxDistance, 0, barWidth - 2);
  if (fillWidth < 0) fillWidth = 0;
  if (fillWidth > barWidth - 2) fillWidth = barWidth - 2;

  display.fillRect(barX + 1, barY + 1, fillWidth, barHeight - 2, SH110X_WHITE);

  // Tính toán giá trị khoảng cách còn lại
  uint32_t remainingDistance = maxDistance - distanceTravelled;
  display.setTextSize(2);

  // Tính chiều rộng thực tế của chuỗi (mỗi ký tự rộng ~16 pixel với textSize=2)
  int digits = remainingDistance == 0 ? 1 : (int)log10(remainingDistance) + 1; // Số chữ số
  int textWidth = digits * 16 + 16; // Chiều rộng của số + "m"
  int maxX = SCREEN_WIDTH - 5; // Giới hạn bên phải màn hình (margin 5 pixel)
  int cursorX = maxX - textWidth; // Dịch sang trái nếu vượt quá giới hạn
  if (cursorX < barX + barWidth - 60) cursorX = barX + barWidth - 60; // Vị trí mặc định

  display.setCursor(cursorX, barY - 15); // Chữ ở trên thanh (52 - 15 = 37)
  display.print(remainingDistance);
  display.print("m");

  Serial.print("Distance Travelled: ");
  Serial.print(distanceTravelled);
  Serial.print(" | Max Distance: ");
  Serial.print(maxDistance);
  Serial.print(" | Remaining: ");
  Serial.print(remainingDistance);
  Serial.print(" | cursorX: ");
  Serial.print(cursorX);
  Serial.print(" | textWidth: ");
  Serial.println(textWidth);
}

// Callback khi có thiết bị kết nối/ngắt kết nối
class MyServerCallbacks: public BLEServerCallbacks {
    void onConnect(BLEServer* pServer) {
      deviceConnected = true;
      Serial.println("Thiết bị đã kết nối");
      display.clearDisplay();
      display.setTextSize(2);
      display.setCursor(20, 20);
      display.println("Connected");
      display.display();
    };

    void onDisconnect(BLEServer* pServer) {
      deviceConnected = false;
      Serial.println("Thiết bị đã ngắt kết nối");
      display.clearDisplay();
      display.setTextSize(2);
      display.setCursor(20, 20);
      display.println("Disconnected");
      display.display();
      BLEDevice::startAdvertising();
    }
};

// Callback khi nhận dữ liệu từ characteristic
class MyCharacteristicCallbacks: public BLECharacteristicCallbacks {
    void onWrite(BLECharacteristic *pCharacteristic) {
      std::string value = pCharacteristic->getValue();
      Serial.print("Dữ liệu mới nhận được, độ dài = ");
      Serial.print(value.length());
      Serial.print(": ");
      
      if (value.length() > 0) {
        for (int i = 0; i < value.length(); i++) {
          char tmp[4];
          sprintf(tmp, "%02X ", value[i]);
          Serial.print(tmp);
        }
        Serial.println();

        display.clearDisplay();
        display.setTextColor(SH110X_WHITE);

        if (value[0] == 0x01 && value.length() >= 3) {
          uint8_t speed = value[1];
          display.setTextSize(2);
          int textWidth = speed < 10 ? 12 : 24;
          int circleX = 16;
          int circleY = 20;
          int xPos = circleX - (textWidth / 2);
          int yPos = circleY - 8;
          display.setCursor(xPos, yPos);
          display.print(speed);
          display.drawCircle(circleX, circleY, 16, SH110X_WHITE);

          uint8_t direction = value[2];
          Serial.print("Direction: 0x");
          Serial.println(direction, HEX);

          if (value.length() > 3) {
            uint32_t distance = 0;
            if (value.length() == 11 && value.substr(3) == "No route") {
              Serial.println("Đã đến đích (No route)");
              drawDestinationIcon(); // Hiển thị chữ "H" khi đến đích
            } 
            else if (value.length() == 7) { // Xử lý "240m"
              std::string distanceStr = value.substr(3, 3);
              distance = atoi(distanceStr.c_str());
              if (distance > 0) {
                if (initialDistance == 0 || distance > initialDistance) {
                  initialDistance = distance;
                  currentDistance = distance;
                } else {
                  currentDistance = distance;
                }
                uint32_t distanceTravelled = initialDistance - currentDistance;
                drawDistanceBar(distanceTravelled, initialDistance);

                if (currentDistance == 0) {
                  Serial.println("Đã đến đích (distance = 0)");
                  drawDestinationIcon(); // Hiển thị chữ "H" khi khoảng cách = 0
                } else {
                  switch (direction) {
                    case 0x08: drawLeftArrow(); Serial.println("Left Arrow"); break;
                    case 0x0A: drawRightArrow(); Serial.println("Right Arrow"); break;
                    case 0x04: drawStraightArrow(); Serial.println("Straight Arrow"); break;
                    default: Serial.println("No arrow drawn"); break;
                  }
                }
              } else {
                Serial.println("Khoảng cách không hợp lệ");
              }
            } 
            else if (value.length() == 8) { // Xử lý "1.7km"
              std::string distanceStr = value.substr(3, 3);
              float distanceKm = atof(distanceStr.c_str());
              distance = (uint32_t)(distanceKm * 1000);
              if (distance > 0) {
                if (initialDistance == 0 || distance > initialDistance) {
                  initialDistance = distance;
                  currentDistance = distance;
                } else {
                  currentDistance = distance;
                }
                uint32_t distanceTravelled = initialDistance - currentDistance;
                drawDistanceBar(distanceTravelled, initialDistance);

                if (currentDistance == 0) {
                  Serial.println("Đã đến đích (distance = 0)");
                  drawDestinationIcon(); // Hiển thị chữ "H" khi khoảng cách = 0
                } else {
                  switch (direction) {
                    case 0x08: drawLeftArrow(); Serial.println("Left Arrow"); break;
                    case 0x0A: drawRightArrow(); Serial.println("Right Arrow"); break;
                    case 0x04: drawStraightArrow(); Serial.println("Straight Arrow"); break;
                    default: Serial.println("No arrow drawn"); break;
                  }
                }
              } else {
                Serial.println("Khoảng cách không hợp lệ");
              }
            } 
            else {
              Serial.println("Dữ liệu khoảng cách không đủ dài hoặc không hỗ trợ");
            }
          } else {
            Serial.println("Không có dữ liệu khoảng cách");
            switch (direction) {
              case 0x08: drawLeftArrow(); Serial.println("Left Arrow"); break;
              case 0x0A: drawRightArrow(); Serial.println("Right Arrow"); break;
              case 0x04: drawStraightArrow(); Serial.println("Straight Arrow"); break;
              default: Serial.println("No arrow drawn"); break;
            }
          }
        }
        display.display();
      } else {
        Serial.println("Không có dữ liệu");
      }
    }
};

void setup() {
  Serial.begin(115200);
  Serial.println("Khởi động ESP32 BLE");

  display.begin(OLED_I2C_ADDRESS, true);
  display.clearDisplay();
  display.setTextSize(2);
  display.setTextColor(SH110X_WHITE);
  display.setCursor(20, 20);
  display.println("Starting...");
  display.display();

  BLEDevice::init("ESP32_Sygic_HUD");
  BLEServer *pServer = BLEDevice::createServer();
  pServer->setCallbacks(new MyServerCallbacks());

  BLEService *pService = pServer->createService(SERVICE_UUID);

  BLECharacteristic *pIndicateCharacteristic = pService->createCharacteristic(
                      CHAR_INDICATE_UUID,
                      BLECharacteristic::PROPERTY_INDICATE
                    );
  pIndicateCharacteristic->addDescriptor(new BLE2902());
  pIndicateCharacteristic->setValue("");

  pWriteCharacteristic = pService->createCharacteristic(
                      CHAR_WRITE_UUID,
                      BLECharacteristic::PROPERTY_WRITE
                    );
  pWriteCharacteristic->setCallbacks(new MyCharacteristicCallbacks());

  pService->start();
  BLEAdvertising *pAdvertising = BLEDevice::getAdvertising();
  pAdvertising->addServiceUUID(SERVICE_UUID);
  pAdvertising->setScanResponse(true);
  pAdvertising->setMinPreferred(0x06);
  pAdvertising->setMinPreferred(0x12);
  BLEDevice::startAdvertising();
  Serial.println("Đang chờ kết nối từ Sygic...");
}

void loop() {
  delay(100);
}