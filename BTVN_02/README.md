# Hướng dẫn cài đặt cảm biến độ ẩm cho trang trại Lan



## 1. Các bước cài đặt

### Bước 1: Cập nhật hệ thống
* **Văn bản giải thích:** Mở Terminal và cập nhật danh sách gói hệ thống để đảm bảo tính tương thích.
* **Câu lệnh Terminal:**
```bash
sudo apt update && sudo apt upgrade -y
```

### Bước 2: Cài đặt thư viện cho cảm biến `DHT22`
* **Văn bản giải thích:** Cài đặt thư viện đọc dữ liệu từ cảm biến.
* **Câu lệnh Terminal:**
```bash
sudo apt install -y git python3-pip
sudo pip3 install Adafruit_DHT
```

### Bước 3: Kích hoạt giao diện 1-Wire
* **Văn bản giải thích:** Bật giao diện 1-Wire trong hệ thống (`Interface Options` \> `1-Wire` \> `Enable`).
* **Câu lệnh Terminal:**
```bash
sudo raspi-config
```

### Bước 4: Kiểm tra module đã được nhận
* **Văn bản giải thích:** Sau khi bật 1-Wire, kiểm tra thiết bị trong thư mục hệ thống.
* **Câu lệnh Terminal:**
```bash
ls /sys/bus/w1/devices/   
```

### Bước 5: Chạy thử chương trình đọc cảm biến
* **Văn bản giải thích:** Tải và chạy file Python mẫu từ thư viện Adafruit.
* **Câu lệnh Terminal:**
```bash
git clone https://github.com/adafruit/Adafruit_Python_DHT.git
cd Adafruit_Python_DHT/examples
python3 AdafruitDHT.py 22 4   
```

### Bước 6: Tự động chạy khi khởi động (Tuỳ chọn)
* **Văn bản giải thích:** Thêm cronjob để đọc dữ liệu định kỳ (ví dụ mỗi 5 phút).
* **Câu lệnh Terminal:**
```bash
crontab -e   
```

---

## 2. Code Python điều khiển máy bơm tự động

Đoạn code sau đây đọc giá trị độ ẩm từ cảm biến `DHT22` qua chân `GPIO 4` và tự động kích hoạt máy bơm (`GPIO 17`) khi độ ẩm xuống dưới mức tiêu chuẩn cho phong lan ($60\%$).

```python
import time
import Adafruit_DHT
import RPi.GPIO as GPIO

# Cấu hình chân phần cứng
SENSOR = Adafruit_DHT.DHT22
DHT_PIN = 4
PUMP_PIN = 17

# Thiết lập GPIO
GPIO.setmode(GPIO.BCM)
GPIO.setup(PUMP_PIN, GPIO.OUT)
GPIO.output(PUMP_PIN, GPIO.LOW)

def control_irrigation():
    try:
        print("Đang đọc cảm biến độ ẩm cho trang trại Lan...")
        while True:
            humidity, temperature = Adafruit_DHT.read_retry(SENSOR, DHT_PIN)
            
            if humidity is not None and temperature is not None:
                print(f"Nhiệt độ: {temperature:.1f}\\u00b0C | Độ ẩm: {humidity:.1f}\\%")
                
                # Ngưỡng độ ẩm lý tưởng cho Lan: 60% - 80%
                if humidity < 60.0:
                    print("Độ ẩm thấp! Bật máy bơm phun sương...")
                    GPIO.output(PUMP_PIN, GPIO.HIGH)
                else:
                    print("Độ ẩm ổn định. Tắt máy bơm.")
                    GPIO.output(PUMP_PIN, GPIO.LOW)
            else:
                print("Lỗi đọc dữ liệu từ cảm biến! Kiểm tra lại kết nối.")
                
            time.sleep(300) # Kiểm tra mỗi 5 phút
    except KeyboardInterrupt:
        print("Dừng chương trình.")
    finally:
        GPIO.cleanup()

if __name__ == "__main__":
    control_irrigation()
```

---

## 3. So sánh loại cảm biến (Cũ vs Mới)

| Tiêu chí | Cảm biến cũ | Cảm biến mới |
| :--- | :--- | :--- |
| **Độ chính xác độ ẩm** | Thấp | Cao |
| **Đo nhiệt độ** | 0C - 50C | 40C - 80C |
| **Tần suất lấy mẫu** | 1 Hz (1 lần/giây) | 0.5 Hz (2 giây/lần) |
| **Độ bền môi trường ẩm** | Kém | Tốt hơn |

---

## 4. Task List kiểm tra thiết bị

* [ ] Kiểm tra nguồn điện cấp cho Raspberry Pi và cảm biến
* [ ] Xác thực kết nối
* [ ] Chạy lệnh thành công không có lỗi gói
* [ ] Kiểm tra kết quả hiển thị trên Terminal đạt ngưỡng
* [ ] Xác nhận máy bơm phản hồi đúng khi độ ẩm xuống dưới $60\\%$