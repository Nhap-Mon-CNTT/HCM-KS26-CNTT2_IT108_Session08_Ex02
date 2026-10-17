# Hướng dẫn cài đặt cảm biến độ ẩm đất cho trang trại Lan

> **Giả định:** Đề bài không cung cấp dữ liệu kỹ thuật, nên thiết bị, chân GPIO, thông số cảm biến và giá tiền trong tài liệu này là **dữ liệu giả định**, chỉ dùng để minh họa cú pháp Markdown.

---

## 1. Giới thiệu

Tài liệu hướng dẫn lắp đặt hệ thống tưới tự động cho **trang trại rau sạch Lan**. Hệ thống gồm 3 luống rau, mỗi luống 1 cảm biến, điều khiển bằng Raspberry Pi 4.

Để tránh nhầm lẫn, tài liệu quy ước như sau:

- Chữ thường là **lời giải thích**.
- Chữ trong khung nền xám như `sudo`, `python3`, `RELAY_PIN` là **tên lệnh hoặc tên biến**.
- Các khối code nhiều dòng là **lệnh cần gõ vào Terminal** hoặc **chương trình Python**.

## 2. Chuẩn bị thiết bị

| Thiết bị | Thông số giả định |
|---|---|
| Bo mạch điều khiển | Raspberry Pi 4 (Raspberry Pi OS) |
| Máy bơm | Bơm mini 5V, điều khiển qua module relay |
| Chân nối relay | `GPIO 17` |
| Chân đọc cảm biến | `GPIO 27` |
| Số lượng cảm biến | 3 (mỗi luống rau 1 cái) |

## 3. Cài đặt phần mềm

Mở Terminal trên Raspberry Pi, lần lượt gõ các lệnh sau. Lệnh `sudo` giúp chạy với quyền quản trị, lệnh `apt` dùng để cài thư viện.

```bash
sudo apt update
sudo apt install python3-rpi.gpio
python3 bom_nuoc.py
```

## 4. Chương trình Python điều khiển máy bơm

Biến `RELAY_PIN` lưu số chân nối với máy bơm, biến `SENSOR_PIN` lưu số chân nối với cảm biến. Chương trình kiểm tra cảm biến mỗi 60 giây, nếu đất khô thì bật bơm 10 giây.

```python
# Chuong trinh dieu khien may bom theo do am dat
import RPi.GPIO as GPIO
import time

RELAY_PIN = 17    # Chan GPIO noi voi relay (may bom)
SENSOR_PIN = 27   # Chan GPIO doc cam bien do am

GPIO.setmode(GPIO.BCM)
GPIO.setup(RELAY_PIN, GPIO.OUT)
GPIO.setup(SENSOR_PIN, GPIO.IN)

try:
    while True:
        if GPIO.input(SENSOR_PIN) == 1:
            print("Dat kho - bat may bom")
            GPIO.output(RELAY_PIN, GPIO.HIGH)
            time.sleep(10)
            GPIO.output(RELAY_PIN, GPIO.LOW)
        else:
            print("Dat du am - khong can tuoi")
        time.sleep(60)
except KeyboardInterrupt:
    print("Dung chuong trinh")
finally:
    GPIO.cleanup()
```

**Giải thích nhanh:**

- `GPIO.OUT`: chân xuất tín hiệu (ra lệnh cho bơm).
- `GPIO.IN`: chân nhận tín hiệu (đọc cảm biến).
- `time.sleep(10)`: chờ 10 giây trong lúc bơm chạy.
- Bấm `Ctrl+C` để dừng chương trình an toàn.

## 5. So sánh hai loại cảm biến

| Tiêu chí | **Loại cũ: FC-28** | **Loại mới: Capacitive v1.2** |
|:---|:---:|:---:|
| Nguyên lý đo | Đo điện trở giữa 2 chân kim loại | Đo thay đổi điện dung |
| Độ chính xác | Trung bình (±10%) | Cao (±3%) |
| Độ bền | Thấp, bị ăn mòn sau 1-3 tháng | Cao, dùng được hơn 2 năm |
| Giá tham khảo | Khoảng 15.000đ | Khoảng 45.000đ |
| Điện áp hoạt động | 3.3V - 5V | 3.3V - 5V |
| Tín hiệu đầu ra | Analog + Digital | Analog |

**Nhận xét:** cảm biến mới đắt hơn khoảng 3 lần nhưng bền và chính xác hơn, phù hợp dùng lâu dài ngoài trời.

## 6. Danh sách kiểm tra thiết bị

- [x] Kiểm tra nguồn điện Raspberry Pi 5V 3A
- [x] Kiểm tra dây nối cảm biến với `GPIO 27`
- [x] Kiểm tra module relay nối `GPIO 17`
- [ ] Kiểm tra máy bơm có nước, ống không bị tắc
- [ ] Đo thử cảm biến trong đất khô và đất ướt
- [ ] Chạy thử script 1 lần và quan sát máy bơm
- [ ] Dán nhãn cảm biến cho luống 1, 2, 3

