# Báo cáo Smart-Farm: phân tích và kiểm thử

## Phần 1. Phân tích Input / Output

### Input

- Số đọc độ ẩm đất từ cảm biến, tín hiệu analog, sau chip `MCP3008` thành giá trị từ 0 đến 1
- Số đọc nhiệt độ từ cảm biến nhiệt độ
- Các tham số do người cài đặt chọn: `NGUONG_KHO`, `THOI_GIAN_BOM`, `CHU_KY_DO`
- Thao tác của người dùng, ví dụ nhấn `Ctrl+C` để dừng chương trình

### Process

Raspberry Pi nhận số đọc, so với `NGUONG_KHO`, quyết định bơm hay không, và ghi một dòng vào nhật ký. Bước này chạy trên CPU của chip SoC theo chu kỳ Fetch, Decode, Execute.

### Output

- Mức điện áp trên `GPIO17`, làm relay đóng hoặc mở
- Máy bơm chạy hoặc dừng
- Dòng nhật ký trong file `smart_farm.log`

### Sơ đồ phối hợp phần cứng

![Sơ đồ hệ thống Smart-Farm: cảm biến độ ẩm và nhiệt độ đi qua chip MCP3008 vào Raspberry Pi, Raspberry Pi điều khiển relay để bật máy bơm](so-do-he-thong.png)

Luồng đi của một lần đo:

1. Cảm biến gửi tín hiệu analog đến `MCP3008`.
2. `MCP3008` đổi sang số và gửi cho Raspberry Pi qua giao tiếp SPI.
3. CPU của Pi so số đọc với ngưỡng.
4. Nếu đất khô, CPU đổi mức điện áp trên `GPIO17`.
5. Relay đóng, nguồn điện đi vào máy bơm.

Relay đứng giữa vì chân GPIO chỉ chịu được dòng rất nhỏ. Nó không đủ sức chạy trực tiếp một máy bơm.

## Phần 2. Triển khai

Sản phẩm là file `README.md` đi kèm ảnh `so-do-he-thong.png`. Cả hai nằm chung một thư mục.

Về hiển thị trên StackEdit: ảnh chỉ hiện khi StackEdit đọc được đường dẫn. Đường dẫn tương đối `so-do-he-thong.png` chạy tốt trên GitHub, nhưng StackEdit cần một địa chỉ công khai. Sau khi đưa ảnh lên GitHub, hãy đổi dòng ảnh trong README thành dạng sau (thay `TEN_BAN`):

```markdown
![Sơ đồ hệ thống Smart-Farm](https://raw.githubusercontent.com/TEN_BAN/smart-farm/main/so-do-he-thong.png)
```

Phần Alt Text nằm trong cặp ngoặc vuông. Khi ảnh lỗi, người đọc thấy dòng chữ đó thay cho ảnh.

## Phần 3. Kiểm thử: RAM đầy dữ liệu nhật ký

### Kịch bản

Một phiên bản chương trình cũ gom nhật ký vào biến danh sách trong RAM và không bao giờ xóa. Sau nhiều ngày chạy, RAM đầy, Pi chậm hoặc đứng.

### Rủi ro chính

Rủi ro lớn nhất không phải là mất nhật ký mà là máy bơm. Nếu Pi đứng đúng lúc relay đang đóng, máy bơm có thể chạy mãi và gây úng nước cho vườn. Vì vậy bước đầu tiên trong mục "Lưu ý" của README là ngắt nguồn bơm bằng tay, trước khi đụng đến bất kỳ lệnh nào.

### Cách xử lý đã đưa vào README

1. Ngắt nguồn máy bơm bằng công tắc riêng hoặc khóa van nước.
2. Kiểm tra bằng `free -h`, `df -h` và `ls -lh smart_farm.log*`.
3. Dừng chương trình bằng `pkill -f bom_tuoi.py`.
4. Chuyển nhật ký cũ sang nơi khác bằng `mv`, không xóa ngay.
5. Khởi động lại bằng `sudo reboot`, không rút điện đột ngột.
6. Chạy lại và theo dõi `free -h` vài phút.

### Cách phòng ngừa

Chương trình ghi nhật ký ra file bằng `RotatingFileHandler`. Mỗi file tối đa 1 MB, giữ thêm 3 file cũ, nên tổng nhật ký không vượt khoảng 4 MB và RAM không bị chiếm dần.

### Các bước kiểm tra sau khi sửa

Hai bước này chưa chạy trên thiết bị thật. Hãy chạy trên Raspberry Pi của bạn rồi ghi kết quả vào đây.

- [ ] Cho chương trình chạy liên tục vài giờ, theo dõi `free -h` mỗi giờ, RAM đã dùng không tăng đều
- [ ] Hạ `maxBytes` xuống một con số nhỏ (ví dụ `1_000`) để ép file xoay vòng, kiểm tra thư mục có `smart_farm.log.1`, `.2`, `.3` và không có file thứ 4
