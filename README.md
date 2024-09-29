### 智慧植物監測系統詳細製作指南

以下將根據您提供的專題資料，詳細介紹如何製作智慧植物監測系統，包括硬體連接、軟體編程、測試與數據分析等步驟。

---

#### **一、所需材料詳述**
1. **Arduino Uno**：微控制器開發板，用於控制感測器並處理數據。
2. **土壤濕度感測器**（如YL-69）：用於測量土壤中的濕度。
3. **光敏電阻（光照感測器）**：用於測量環境光照強度。
4. **SD卡模組**（如Catalex Micro SD Card Module）：用於將數據儲存到SD卡。
5. **SD卡**：存儲數據，建議使用Class 10或更高等級的SD卡。
6. **連接線**：跳線，用於連接各模組與Arduino。
7. **面包板**：臨時連接電路，方便調試。
8. **電阻**：光敏電阻需要電阻分壓，通常使用10kΩ電阻。
9. **USB線**：連接Arduino至電腦，用於上傳程式碼和串口監控。

---

#### **二、硬體連接詳解**

1. **土壤濕度感測器連接**
   - **VCC** 接 Arduino 的 **5V**。
   - **GND** 接 Arduino 的 **GND**。
   - **SIG（信號腳）** 接 Arduino 的 **A0（模擬輸入0）**。

2. **光敏電阻（光照感測器）連接**
   - 將光敏電阻的一端接至 Arduino 的 **5V**。
   - 另一端與 **10kΩ電阻** 連接，形成分壓電路。
   - 連接點（光敏電阻與電阻之間）接至 Arduino 的 **A1（模擬輸入1）**。
   - **10kΩ電阻** 的另一端接至 **GND**。

3. **SD卡模組連接**
   - **VCC** 接 Arduino 的 **5V**。
   - **GND** 接 Arduino 的 **GND**。
   - **MOSI** 接 Arduino 的 **Pin 11**。
   - **MISO** 接 Arduino 的 **Pin 12**。
   - **SCK** 接 Arduino 的 **Pin 13**。
   - **CS（Chip Select）** 接 Arduino 的 **Pin 10**。（可根據實際模組設定）

4. **電路總覽圖**
   ```
   Arduino Uno
   +-------------------+
   |                   |
   | 5V      GND       |
   | |        |        |
   | |        |        |
   | |        |        |
   | A0       A1        |
   | |        |        |
   | |        |        |
   | Pin10  Pin11 Pin12 Pin13
   | |        |     |      |
   | |        |     |      |
   | SD 模組  接線連接   |
   |                   |
   +-------------------+
   
   土壤濕度感測器和光敏電阻分別接至A0和A1
   ```

---

#### **三、Arduino 程式碼編寫**

以下提供一個範例程式碼，實現定時讀取濕度和光照數據，並將其儲存到SD卡，同時可選擇性串口輸出數據。

```cpp
#include <SPI.h>
#include <SD.h>

// 定義感測器腳位
const int soilMoisturePin = A0;
const int lightSensorPin = A1;

// SD卡片選腳位
const int chipSelect = 10;

// 設定讀取間隔（毫秒）
const unsigned long interval = 3600000; // 每小時一次
unsigned long previousMillis = 0;

File dataFile;

void setup() {
  // 啟動序列通訊，用於偵錯
  Serial.begin(9600);
  
  // 初始化感測器腳位
  pinMode(soilMoisturePin, INPUT);
  pinMode(lightSensorPin, INPUT);
  
  // 初始化SD卡模組
  if (!SD.begin(chipSelect)) {
    Serial.println("SD卡初始化失敗！");
    return;
  }
  Serial.println("SD卡初始化成功。");
  
  // 打開或創建數據文件，附加模式
  dataFile = SD.open("datalog.txt", FILE_WRITE);
  if (dataFile) {
    dataFile.println("時間,土壤濕度,光照強度");
    dataFile.close();
    Serial.println("數據文件創建/打開成功。");
  } else {
    Serial.println("無法打開數據文件。");
  }
}

void loop() {
  unsigned long currentMillis = millis();
  
  if (currentMillis - previousMillis >= interval) {
    previousMillis = currentMillis;
    
    // 讀取感測器數據
    int soilMoistureValue = analogRead(soilMoisturePin);
    int lightSensorValue = analogRead(lightSensorPin);
    
    // 獲取當前時間（此處僅簡單計數，實際可使用RTC模組）
    unsigned long timestamp = currentMillis / 1000 / 60 / 60; // 小時數
    
    // 打開數據文件進行寫入
    dataFile = SD.open("datalog.txt", FILE_WRITE);
    if (dataFile) {
      dataFile.print(timestamp);
      dataFile.print(",");
      dataFile.print(soilMoistureValue);
      dataFile.print(",");
      dataFile.println(lightSensorValue);
      dataFile.close();
      Serial.print("數據已儲存：");
      Serial.print(timestamp);
      Serial.print("小時, ");
      Serial.print(soilMoistureValue);
      Serial.print(", ");
      Serial.println(lightSensorValue);
    } else {
      Serial.println("無法打開數據文件進行寫入。");
    }
  }
}
```

**程式碼說明：**
- 引入 `SPI` 和 `SD` 庫以支援SD卡模組。
- 定義感測器接腳及SD卡片選腳位。
- 在 `setup()` 中初始化序列通訊和SD卡，並創建數據文件 `datalog.txt`，首行寫入標題。
- 在 `loop()` 中，每隔指定時間（例：1小時）讀取感測器數據，並將時間戳、土壤濕度和光照強度寫入SD卡，同時透過序列監控輸出。

**注意事項：**
- 由於Arduino Uno沒有實時時鐘（RTC），此範例使用了簡單的 `millis()` 計數來模擬時間。若需更準確的時間戳，建議添加RTC模組（如DS1307或DS3231）。
- 確保 `datalog.txt` 文件不會超過SD卡的存儲限制，定期備份和清理數據是必要的。

---

#### **四、硬體組裝步驟**

1. **連接Arduino與面包板**
   - 使用跳線將Arduino Uno放置於面包板旁，確保組件之間方便連接。

2. **接線土壤濕度感測器**
   - 將土壤濕度感測器的 **VCC** 和 **GND** 分別接至Arduino的 **5V** 和 **GND**。
   - 將 **SIG** 腳接至Arduino的 **A0**。

3. **接線光敏電阻**
   - 將光敏電阻的一端接至Arduino的 **5V**。
   - 另一端連接至 **10kΩ電阻**，中間點接至 **A1**，電阻的另一端接至 **GND**。

4. **接線SD卡模組**
   - 將SD卡模組的 **VCC** 和 **GND** 接至Arduino的 **5V** 和 **GND**。
   - 將 **MOSI**、**MISO**、**SCK** 和 **CS** 分別接至Arduino的 **Pin 11**、**Pin 12**、**Pin 13** 和 **Pin 10**。

5. **檢查連接**
   - 確認所有連接無誤，避免短路或接錯腳位。

---

#### **五、系統測試與校正**

1. **上傳程式碼**
   - 使用Arduino IDE將上述程式碼上傳至Arduino Uno。

2. **監控串口輸出**
   - 開啟Arduino IDE的串口監控器，設置波特率為 **9600**。
   - 確認SD卡是否初始化成功，並觀察數據寫入情況。

3. **檢查SD卡內容**
   - 上傳程式碼後，拔下SD卡並插入電腦，檢查 `datalog.txt` 文件是否正確記錄數據。
   - 每小時確認一次記錄是否新增新數據。

4. **校正感測器**
   - **土壤濕度感測器**：
     - 在不同濕度條件下（乾燥、濕潤）測量感測器的輸出值，並記錄對應的濕度。
     - 可根據測試數據調整水分補給頻率。
   - **光敏電阻**：
     - 在不同光照條件（強光、昏暗）下測量感測器的輸出值，校正光照強度的讀數。

---

#### **六、數據整理與分析**

1. **從SD卡導出數據**
   - 將SD卡插入電腦，打開 `datalog.txt` 文件。
   - 數據格式示例：
     ```
     時間,土壤濕度,光照強度
     1,500,300
     2,480,320
     ...
     ```

2. **使用Excel進行數據分析**
   - 將 `datalog.txt` 以逗號分隔（CSV格式）匯入Excel。
   - 檢查數據完整性，確保沒有缺失或異常值。
   - **繪製圖表**：
     - **土壤濕度變化圖**：時間（X軸） vs. 濕度（Y軸）。
     - **光照強度變化圖**：時間（X軸） vs. 光照（Y軸）。
     - **相關性分析**：土壤濕度與光照之間的關聯。

3. **進階分析（可選）**
   - 使用統計軟體（如Python的Pandas、Matplotlib）進行更深入的數據分析。
   - 預測植物生長趨勢，優化灌溉和光照管理。

---

#### **七、系統優化與擴展（可選）**

1. **實時監控**
   - 修改程式碼，持續將數據通過串口傳輸至電腦，使用串口監控工具（如Serial Plotter）即時顯示數據。

2. **遠端數據傳輸**
   - 添加無線模組（如ESP8266、ESP32）將數據上傳至雲端平台（如ThingSpeak、Blynk），實現遠端實時監控。

3. **自動化控制**
   - 根據濕度和光照數據，自動控制灌溉系統或LED燈，以優化植物生長環境。

4. **增加更多感測器**
   - 如溫度和濕度感測器（DHT22）、CO₂感測器等，全面監控植物生長環境。

---

### **總結**

透過以上詳細步驟，您可以成功製作一個簡單且實用的智慧植物監測系統。此系統不僅能實時監控植物生長環境，還能累積數據為後續分析提供依據。根據需求，您可進一步擴展系統功能，提升系統的智能化程度。

如在製作過程中遇到問題，建議參考Arduino官方文件、相關感測器模組的數據手冊，或在Arduino社群中尋求協助。祝您的專題順利完成！
