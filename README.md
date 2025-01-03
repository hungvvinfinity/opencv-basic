# opencv-basic
### Ảnh được biểu diễn như thế nào?
dưới dạng ma trận. Mỗi phần tử trong ma trận này tương ứng với một điểm ảnh (pixel) trong ảnh
đây là ví dụ đơn giản về biểu diễn ảnh bằng ma trận
![img](sample-matrix.gif)

Cách Biểu Diễn Ảnh
 1. Ma Trận Một Kênh (Grayscale) từ 0 đến 255, trong đó 0 là màu đen và 255 là màu trắng
 2. Ma Trận Ba Kênh (Color) [Blue, Green, Red] => [0, 0, 255] đây là màu đỏ
 3. Ma Trận Bốn Kênh (Transparency) [B, G, R, A] => đỏ 50% tranparent [0, 0, 255, 128]

### Thao Tác Cơ Bản Với Ảnh
#### 1. Chuyển đổi sang ảnh xám
```python
cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
```
tại sao hàm cơ bản này quan trọng?
- chuyển về 1 kênh màu so với ba kênh trong ảnh RGB, làm giảm đáng kể bộ nhớ và tốc độ xử lý
- trong 1 số bài toán hoạt động tốt hơn trên ảnh xám vì không có sự phân tán của thông tin màu sắc, vd: Phát Hiện Cạnh và Đặc Trưng

#### 2. Cắt ảnh: 
```python
cropped_img = img[y:y+h, x:x+w]
```
vd: y = 3, x = 5
cắt ảnh 2x2 => img[3:3+2, 5:5+2]

#### 3. Làm mờ ảnh: 
```python
blurred_img = cv2.GaussianBlur(img, (k, k), 0)
```
minh họa kernel Gaussian phân bố trọng số xung quanh tâm
![kernel](https://miro.medium.com/v2/resize:fit:828/format:webp/1*Nf8jVYj2zhPPOjJSQrY9Ug.png)

k: kernel size, kích thước này phải là số lẻ để có một điểm trung tâm
chú ý tổng các trọng số phải = 1
Nếu tổng lớn hơn 1: Ảnh kết quả sẽ sáng hơn ảnh gốc.
Nếu tổng nhỏ hơn 1: Ảnh kết quả sẽ tối hơn ảnh gốc.

sigmaX: "độ rộng" của hình chuông Gaussian, sigma càng lớn làm mờ càng mạnh

GaussianBlur hoạt động như thế nào: về cơ bản thi nó replace từng pixel bằng weight trung bình của các pixel xung quanh nó

[Gaussian Blurring — A Gentle Introduction](https://pub.towardsai.net/gaussian-blurring-a-gentle-introduction-e34aca1d9bbd)
![minh họa](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*Ra4DG6PT0hxnvH2aW2OUKw.gif)

vậy các góc của ma trận không đủ để tính làm mờ thì sao?
Zero padding: Thêm các điểm ảnh với giá trị 0 xung quanh ảnh
Replicate padding: chưa hiểu lắm, :)
Reflect padding: Lặp lại ảnh theo chiều đối xứng ở các biên

* ngoài ra còn có nhiều phương pháp làm mờ khác nữa

  - Box Blur: kernel dạng box (vuông hoặc chữ nhật) mỗi điểm ảnh trong khu vực kernel sẽ có ảnh hưởng như nhau
  - Median Blur: nó dùng trung vị để xác định giá trị trung tâm mới, [ví dụ](https://docs.gimp.org/en/gimp-filter-median-blur.html) ma trận 3x3 sắp xếp từ bé đến lớn, trung vị là giá trị ở giữa -> thay thế cho điểm trung tâm từ 161 -> 186
  - Bilateral Filter: Làm mờ ảnh bằng cách sử dụng hai hàm Gaussian, một cho không gian và một cho cường độ màu
  .v.v. nhiều vãi ae tự tiềm hiểu gpt ra 1 rổ

 **ứng dụng trong các app làm mịn da**
 ```python
# Gaussian Blur
smoothed = cv2.GaussianBlur(image, (5, 5), 0)

# Median Blur
smoothed = cv2.medianBlur(image, 5)

# Làm mịn da mà giữ nguyên cạnh
bilateral_filtered = cv2.bilateralFilter(image, 9, 75, 75)
```

#### 4. Phát hiện cạnh và viền: 
```python
edges = cv2.Canny(image, threshold1 = 100, threshold2 = 200)
```
Trong ví dụ trên ngưỡng dưới là 100 và ngưỡng trên là 200, để liên kết các cạnh yếu với cạnh mạnh.

![edge](https://wisdomml.in/wp-content/uploads/2023/02/canny1.png)
tôi hiểu đơn giản
cạnh là các điểm trong ảnh được highlight trắng trong nền đen, có thể đứt đoạn

Đường Viền: đường viền đóng (liên tục), mô tả hình dáng của vật thể, đường viền thường dựa trên cạnh để xác định ranh giới của đối tượng
```python
contours, _ = cv2.findContours(edges, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
```
**ứng dụng trong các app kiểu phát hiện góc tài liệu để khi chụp tự động cắt góc**

#### 5. Cân bằng sáng, tương Phản, chỉnh màu, tăng cường màu
```python
# Điều chỉnh độ sáng và tương phản
# alpha là hệ số tương phản, beta là giá trị tăng giảm độ sáng
adjusted = cv2.convertScaleAbs(image, alpha=1.5, beta=50)

# cân bằng kênh
# Chuyển đổi ảnh sang không gian màu LAB
lab_image = cv2.cvtColor(image, cv2.COLOR_BGR2Lab)
# Tách các kênh L, A, và B
l, a, b = cv2.split(lab_image)
# Cân bằng kênh L
l_equalized = cv2.equalizeHist(l)

# tăng cường màu
# Chuyển đổi ảnh sang không gian màu HSV
hsv_image = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)
# Tăng cường màu xanh
mask = cv2.inRange(hsv_image, (60, 100, 100), (180, 255, 255))
hsv_image[mask > 0] = [120, 255, 255]
```

**các template, filter trong các app ảnh**


