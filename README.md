# Găng Tay Dịch Ngôn Ngữ Ký Hiệu Thông Minh

Dự án này sử dụng ESP32, cảm biến uốn cong (flex sensors), cảm biến MPU6050 (gia tốc kế và con quay hồi chuyển) và loa MAX98357A để dịch ngôn ngữ ký hiệu sang văn bản và phát ra âm thanh tiếng Việt thông qua dịch vụ VoiceRSS.

## Mục lục
1.  [Yêu cầu Phần cứng](#1-yêu-cầu-phần-cứng)
2.  [Chuẩn bị Môi trường Phần mềm](#2-chuẩn-bị-môi-trường-phần-mềm)
    * [Cài đặt Arduino IDE](#cài-đặt-arduino-ide)
    * [Cài đặt ESP32 Board Package](#cài-đặt-esp32-board-package)
    * [Cài đặt Thư viện](#cài-đặt-thư-viện)
    * [Công cụ Tải Hệ thống Tệp LittleFS](#công-cụ-tải-hệ-thống-tệp-littlefs)
3.  [Nạp Code và Dữ liệu lên ESP32](#3-nạp-code-và-dữ-liệu-lên-esp32)
    * [Nạp Code C++ (Sketch)](#nạp-code-c-sketch)
    * [Tải Hệ thống Tệp (Web Files)](#tải-hệ-thống-tệp-web-files)
4.  [Kết nối và Sử dụng Găng Tay](#4-kết-nối-và-sử-dụng-găng-tay)
    * [Khởi động ESP32](#khởi-động-esp32)
    * [Mở Serial Monitor](#mở-serial-monitor)
    * [Truy cập Giao diện Web](#truy-cập-giao-diện-web)
    * [Sử dụng Giao diện](#sử-dụng-giao-diện)
    * [Huấn luyện Cử chỉ](#huấn-luyện-cử-chỉ)
5.  [Gỡ lỗi Cơ bản](#5-gỡ-lỗi-cơ-bản)

---

## 1. Yêu cầu Phần cứng

* **Bo mạch ESP32:** Bất kỳ bo mạch phát triển ESP32 nào có đủ chân GPIO.
* **Cảm biến Uốn cong (Flex Sensors):** 05 chiếc.
* **Điện trở 4.7k Ohm:** 05 chiếc (cho mỗi cảm biến uốn cong, tạo thành mạch chia áp).
* **Cảm biến MPU6050:** Gia tốc kế và con quay hồi chuyển.
* **Loa MAX98357A I2S Amplifier:** Và một loa nhỏ tương thích.
* **Dây nối, Breadboard (nếu cần):** Để kết nối các linh kiện.
* **Nguồn cấp:** Qua cổng USB hoặc nguồn ngoài phù hợp cho ESP32.
* **Cáp USB:** Loại tốt, có khả năng truyền dữ liệu.

**Kết nối:** Đảm bảo tất cả các cảm biến và loa được kết nối chính xác với các chân GPIO trên ESP32 theo sơ đồ mạch của dự án và định nghĩa chân trong code C++.
    * Loa MAX98357A: DIN -> GPIO 25, BCLK -> GPIO 26, LRC -> GPIO 27.
    * Cảm biến uốn cong: Kết nối với các chân ADC1 (ví dụ: 32, 33, 34, 35, 36, 39).
    * MPU6050: Kết nối qua I2C (SDA, SCL).

---

## 2. Chuẩn bị Môi trường Phần mềm

### Cài đặt Arduino IDE
* Tải và cài đặt phiên bản Arduino IDE mới nhất từ [arduino.cc](https://www.arduino.cc/en/software).

### Cài đặt ESP32 Board Package
1.  Mở Arduino IDE, vào **File > Preferences**.
2.  Trong ô "Additional Board Manager URLs", thêm URL sau:
    ```
    [https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json](https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json)
    ```
3.  Nhấn **OK**.
4.  Vào **Tools > Board > Boards Manager...**.
5.  Tìm kiếm "esp32" và cài đặt "esp32 by Espressif Systems".
6.  Sau khi cài đặt, chọn đúng bo mạch ESP32 bạn đang sử dụng trong **Tools > Board** (ví dụ: "ESP32 Dev Module").

### Cài đặt Thư viện
1.  Trong Arduino IDE, vào **Sketch > Include Library > Manage Libraries...**.
2.  Tìm và cài đặt các thư viện sau (nhập tên thư viện vào ô tìm kiếm và nhấn Install):
    * `Adafruit MPU6050` (bởi Adafruit)
    * `Adafruit Unified Sensor` (bởi Adafruit - thường được cài đặt cùng với MPU6050)
    * `ESPAsyncWebServer` (Tìm kiếm "ESPAsyncWebServer". Nếu không thấy, bạn có thể cần tải file ZIP từ GitHub của tác giả me-no-dev và cài đặt thủ công qua Sketch > Include Library > Add .ZIP Library...).
    * `AsyncTCP` (Thường là thư viện phụ thuộc của ESPAsyncWebServer và sẽ được nhắc cài đặt cùng).
    * `Arduino_JSON` (bởi Arduino)

### Công cụ Tải Hệ thống Tệp LittleFS
Để tải các file giao diện web (HTML, CSS) lên bộ nhớ flash của ESP32, bạn cần plugin LittleFS cho Arduino IDE.
1.  Truy cập trang GitHub của plugin: [ESP32/ESP8266 LittleFS Filesystem Uploader](https://github.com/lorol/LITTLEFS).
2.  Làm theo hướng dẫn cài đặt trong phần "Installation" của trang đó. Thông thường, bạn sẽ tải file `.zip` của plugin, giải nén và sao chép thư mục `ESP32FS` (hoặc `LITTLEFS`) vào thư mục `tools` trong thư mục Sketchbook của Arduino IDE (bạn có thể tìm đường dẫn thư mục Sketchbook trong File > Preferences).
3.  Khởi động lại Arduino IDE. Sau đó, bạn sẽ thấy tùy chọn "ESP32 Sketch Data Upload" trong menu "Tools".

---

## 3. Nạp Code và Dữ liệu lên ESP32

### Nạp Code C++ (Sketch)
1.  Mở file code C++ của dự án (ví dụ: `sign_language_glove_arduino_no_physical_buttons_gyro_fix_final.ino`) bằng Arduino IDE.
2.  **Cấu hình Quan trọng trong Code:**
    * **WiFi Credentials:**
        ```cpp
        const char *ssid = "Kien";     // THAY BẰNG TÊN WIFI CỦA BẠN
        const char *password = "0968404490"; // THAY BẰNG MẬT KHẨU WIFI CỦA BẠN
        ```
    * **VoiceRSS API Key:**
        ```cpp
        #define VOICERSS_API_KEY "4191561eb8d34538a2dc0d746c5e14d5" // API Key của bạn
        ```
    * **(Rất quan trọng) Hiệu chỉnh Cảm biến Uốn cong:** Trong hàm `loop()`, tìm đến phần được đánh dấu:
        ```cpp
        // !!! QUAN TRỌNG: THAY THẾ CÁC GIÁ TRỊ NÀY BẰNG GIÁ TRỊ THỰC NGHIỆM CỦA BẠN !!!
        // Đây chỉ là ví dụ, bạn PHẢI tự đo cho từng cảm biến với điện trở 4.7k Ohm.
        // Ghi lại giá trị analogRead() thấp nhất (thẳng) và cao nhất (cong) cho mỗi ngón.
        int adc_min_straight_thumb = 400;  int adc_max_bent_thumb = 2800;  // THAY THẾ!
        int adc_min_straight_index = 450;  int adc_max_bent_index = 2900;  // THAY THẾ!
        int adc_min_straight_middle= 420;  int adc_max_bent_middle= 2750; // THAY THẾ!
        int adc_min_straight_ring  = 480;  int adc_max_bent_ring  = 3000;  // THAY THẾ!
        int adc_min_straight_pinky = 500;  int adc_max_bent_pinky = 3100;  // THAY THẾ!
        ```
        **Cách hiệu chỉnh:** Mở Serial Monitor (sau khi đã nạp code lần đầu với các giá trị tạm), giữ một ngón tay thẳng hoàn toàn và quan sát giá trị `Raw Flex` tương ứng trên Serial Monitor (code đã có sẵn các dòng `Serial.print` để hỗ trợ việc này). Ghi lại giá trị thấp nhất bạn thấy cho ngón đó. Sau đó, uốn cong ngón tay đó hết mức và ghi lại giá trị `Raw Flex` cao nhất. Lặp lại cho tất cả 5 ngón tay và cập nhật các giá trị `adc_min_straight_...` và `adc_max_bent_...` này vào code.
3.  **Chọn Cổng COM:** Trong Arduino IDE, vào **Tools > Port** và chọn đúng cổng COM mà ESP32 của bạn đang kết nối.
4.  **Nạp Sketch:** Nhấn nút "Upload" (biểu tượng mũi tên sang phải) để biên dịch và nạp sketch lên ESP32.

### Tải Hệ thống Tệp (Web Files)
1.  Trong thư mục chứa sketch Arduino của bạn (ví dụ: `sign_language_glove_project`), tạo một thư mục con tên là `data` (nếu chưa có).
2.  Sao chép các file giao diện web sau vào thư mục `data` này:
    * `index.html` (ID: `sign_language_translator_html_new_ui`)
    * `style.css` (ID: `sign_language_translator_css_new_ui`)
    * **Lưu ý:** Không cần file `main.js` hay `hand.glb` cho phiên bản giao diện không có mô hình 3D.
3.  Trong Arduino IDE, vào **Tools > ESP32 Sketch Data Upload**. Quá trình này sẽ tải tất cả các file trong thư mục `data` lên hệ thống tệp LittleFS của ESP32. Nếu bạn cập nhật các file web, bạn cần thực hiện lại bước này.

---

## 4. Kết nối và Sử dụng Găng Tay

### Khởi động ESP32
* Sau khi đã nạp code và dữ liệu hệ thống tệp, ngắt kết nối ESP32 khỏi máy tính rồi cắm lại nguồn (hoặc nhấn nút Reset trên ESP32).

### Mở Serial Monitor
1.  Trong Arduino IDE, mở **Serial Monitor** (Tools > Serial Monitor hoặc biểu tượng kính lúp).
2.  Đặt tốc độ baud là **115200**.
3.  Quan sát các thông báo khởi động. ESP32 sẽ cố gắng kết nối vào mạng WiFi bạn đã khai báo.
4.  Sau khi kết nối WiFi thành công, bạn sẽ thấy dòng chữ:
    ```
    Đã kết nối WiFi
    Địa chỉ IP: XXX.XXX.XXX.XXX 
    ```
    Hãy ghi lại **Địa chỉ IP** này. Đây chính là địa chỉ để truy cập giao diện web của găng tay.
5.  Bạn cũng sẽ thấy quá trình hiệu chỉnh con quay hồi chuyển diễn ra ("Đang hiệu chỉnh con quay hồi chuyển..."). **Hãy giữ găng tay hoàn toàn đứng yên trên một bề mặt phẳng trong lúc này.**
6.  Sau khi hiệu chỉnh xong, bạn sẽ nghe một âm thanh ngắn từ loa (nếu đã kết nối đúng) và thấy thông báo "Thiết lập hoàn tất...".

### Truy cập Giao diện Web
1.  Mở một trình duyệt web (Chrome, Firefox, Edge, v.v.) trên máy tính hoặc điện thoại đang kết nối **cùng mạng WiFi** với ESP32.
2.  Trên thanh địa chỉ của trình duyệt, nhập địa chỉ IP bạn đã ghi lại ở bước trên (ví dụ: `http://192.168.1.10`). Nhấn Enter.
3.  Giao diện web của găng tay dịch ngôn ngữ ký hiệu sẽ được tải lên.

### Sử dụng Giao diện
* **Bảng Điều Khiển:**
    * **Trạng thái:** Cho biết tình trạng kết nối với ESP32.
    * **Dự đoán:** Hiển thị kết quả dịch cử chỉ.
    * **Bắt đầu Dự đoán:** Nhấn nút này để ESP32 bắt đầu thu thập dữ liệu và dự đoán cử chỉ bạn thực hiện. Sau khi nhấn, hãy thực hiện cử chỉ trong khoảng 3 giây.
    * **Đặt lại Gyro:** Nhấn nút này nếu bạn thấy giá trị Gyro trên giao diện bị trôi (tăng/giảm liên tục dù không di chuyển). Nút này sẽ reset giá trị gyro tích lũy trên ESP32 về 0.
    * **Tắt tiếng/Bật tiếng:** Điều khiển việc phát âm thanh TTS từ ESP32 (qua VoiceRSS).
    * **Hiện/Ẩn Bảng Huấn luyện:** Chuyển đổi hiển thị của bảng huấn luyện.
* **Thông số Cảm biến:** Hiển thị giá trị đọc được từ các cảm biến uốn cong, con quay hồi chuyển (gyro), và gia tốc kế (accel) theo thời gian thực.
* **Chế độ Huấn luyện:**
    * **Nhãn Cử chỉ:** Nhập tên (bằng tiếng Việt, không dấu phẩy) cho cử chỉ bạn muốn dạy găng tay (ví dụ: "xin chao", "cam on").
    * **Bắt đầu Ghi Cử chỉ:** Sau khi nhập nhãn, nhấn nút này. Ngay sau đó, hãy thực hiện cử chỉ bạn muốn ghi một cách rõ ràng và nhất quán trong khoảng 3 giây. ESP32 sẽ thu thập dữ liệu và lưu lại vào file `SensorData.csv` trên bộ nhớ của nó.
    * **Phản hồi:** Cho biết trạng thái của quá trình huấn luyện (ví dụ: "Đã lưu 'xin chao'!").

### Huấn luyện Cử chỉ
* Để hệ thống dự đoán chính xác, bạn cần huấn luyện cho nó các cử chỉ.
* Với mỗi cử chỉ, hãy thực hiện việc ghi lại (huấn luyện) **nhiều lần** (ví dụ: 5-10 lần, hoặc nhiều hơn nếu cần) để mô hình có thể học được sự đa dạng trong cách bạn thực hiện cử chỉ.
* Đảm bảo thực hiện cử chỉ một cách nhất quán trong quá trình huấn luyện.
* Sau khi huấn luyện, hãy thử chế độ "Bắt đầu Dự đoán" để xem găng tay có nhận diện đúng không. Nếu dự đoán chưa chính xác, hãy thử huấn luyện thêm cho cử chỉ đó hoặc kiểm tra lại việc hiệu chỉnh cảm biến uốn cong.

---

## 5. Gỡ lỗi Cơ bản
* **Không có gì trên Serial Monitor:** Kiểm tra lại cổng COM, tốc độ baud, cáp USB, nguồn cấp cho ESP32. Thử tải một sketch "Blink" đơn giản.
* **Không kết nối được WiFi:** Kiểm tra lại tên SSID và mật khẩu WiFi trong code C++. Đảm bảo ESP32 ở trong vùng phủ sóng WiFi.
* **Giao diện web không tải được:** Kiểm tra lại địa chỉ IP của ESP32. Đảm bảo máy tính/điện thoại của bạn đang kết nối cùng mạng WiFi với ESP32. Kiểm tra xem các file HTML/CSS đã được tải lên thư mục `data` và lên ESP32 đúng cách chưa.
* **Dự đoán không chính xác:**
    * **QUAN TRỌNG NHẤT:** Kiểm tra lại việc **Hiệu chỉnh Cảm biến Uốn cong**. Đây là nguyên nhân phổ biến nhất.
    * Huấn luyện thêm dữ liệu cho các cử chỉ.
    * Đảm bảo các cử chỉ huấn luyện đủ khác biệt nhau.
* **Gyro bị trôi nhiều:**
    * Đảm bảo bạn giữ găng tay thật yên trong quá trình `calibrateMPU()` khi khởi động.
    * Sử dụng nút "Đặt lại Gyro" trên giao diện web thường xuyên.
    * Có thể thử tinh chỉnh nhẹ các giá trị `gyroXerror`, `gyroYerror`, `gyroZerror` trong code C++.

Chúc bạn thành công với dự án!
