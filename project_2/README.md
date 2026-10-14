# Phần 2.2 - Low-pass và High-pass Filter

## 1. Chương trình làm gì?

Chương trình thực hiện đúng luồng xử lý ảnh cơ bản:

```text
Ảnh đầu vào -> Đọc bằng OpenCV -> Áp dụng bộ lọc -> Lưu ảnh đầu ra
```

Các ảnh trong thư mục `images/` được xử lý độc lập. Bộ ảnh mẫu hiện có gồm:

- `urban.png`: ảnh có nhiều cạnh và chi tiết kiến trúc.
- `still_life.png`: ảnh đơn giản, có nhiều vùng phẳng.
- `Landscape-fabric-under-gravel.png`: ảnh có nhiều lá, sỏi và texture nhỏ.

Đây là ba ảnh riêng cho Phần 2, không phải đầu ra của Phần 1.

## 2. Cách chạy

```bash
cd project_2
pip install -r requirements.txt
jupyter notebook filter_part2.ipynb
```

Ảnh đầu ra được lưu trực tiếp trong `results/`. Với mỗi ảnh đầu vào, chương trình tạo:

```text
<ten>_mean.png
<ten>_gaussian.png
<ten>_laplacian.png
<ten>_sobel.png
<ten>_comparison.png
metrics.csv (MSE và edge energy)
urban_custom.png (kernel custom, áp dụng riêng lên urban.png)
urban_custom_comparison.png (so sánh ảnh gốc và ảnh custom)
```

Mỗi lần chạy, chương trình tự xóa các ảnh kết quả do chính nó tạo ở lần trước, sau đó chỉ sinh kết quả cho các ảnh đang có trong `images/`. Ví dụ: lần đầu input là `A, B, C` thì output là `A', B', C'`; nếu thay `A` bằng `D` thì lần chạy sau output chỉ còn `D', B', C'`. Các file khác bạn tự đặt trong `results/` sẽ không bị xóa.

Bạn có thể xóa ảnh mẫu và chép vào `images/` bất kỳ số lượng ảnh PNG, JPG, JPEG hoặc BMP. Chương trình tự quét thư mục, nên 2, 3, 4 hoặc nhiều ảnh hơn đều được xử lý mà không cần sửa code. File `source_contact_sheet.png` (nếu còn) là ảnh nguồn trung gian và được chương trình bỏ qua.

## 3. Low-pass filter

Low-pass filter làm trơn ảnh bằng cách kết hợp giá trị của các pixel lân cận. Nó làm giảm nhiễu và chi tiết nhỏ, nhưng cũng làm cạnh bị mờ.

### Mean filter 5x5

Lệnh sử dụng:

```python
mean = cv2.blur(image, (5, 5))
```

Kernel tương ứng là:

```text
H = 1/25 * [1 1 1 1 1
            1 1 1 1 1
            1 1 1 1 1
            1 1 1 1 1
            1 1 1 1 1]
```

Mỗi pixel đầu ra là trung bình của 25 pixel trong vùng 5x5. Tổng các hệ số bằng 1 nên độ sáng trung bình của ảnh gần như được giữ nguyên.

Ưu điểm: dễ hiểu, tính toán nhanh. Nhược điểm: tất cả pixel có trọng số bằng nhau nên đường biên dễ bị nhòe.

### Gaussian filter 5x5

Lệnh sử dụng:

```python
gaussian = cv2.GaussianBlur(image, (5, 5), 0)
```

Gaussian cũng lấy trung bình các pixel lân cận, nhưng pixel gần tâm có trọng số lớn hơn pixel ở xa. Vì vậy ảnh thường được làm trơn tự nhiên hơn Mean filter. Tham số `0` cho phép OpenCV tự chọn sigma từ kích thước kernel.

## 4. Metric để so sánh Mean và Gaussian

Chương trình ghi kết quả vào `results/metrics.csv`. Có hai metric đơn giản, cùng được tính trên ảnh xám để việc so sánh công bằng:

### MSE so với ảnh gốc

Với ảnh gốc `I` và ảnh sau lọc `F`, trước tiên chuẩn hóa pixel về `[0, 1]`. Công thức là:

```text
MSE(I, F) = 1/(H*W) * sum_y sum_x (I(x,y) - F(x,y))^2
```

Trong đó `H` và `W` là chiều cao và chiều rộng ảnh. MSE đo mức ảnh sau lọc đã thay đổi bao nhiêu so với ảnh gốc:

- MSE bằng 0 ở chính ảnh gốc.
- MSE càng lớn nghĩa là bộ lọc làm thay đổi ảnh càng nhiều.
- MSE không phải điểm chất lượng tuyệt đối. Không có ảnh chuẩn không nhiễu trong thí nghiệm nên không thể nói MSE nhỏ luôn tốt hơn.

### Edge energy

Để đo lượng biên và chi tiết còn lại, chương trình tính gradient Sobel trên ảnh xám đã chuẩn hóa:

```text
E(F) = 1/(H*W) * sum_y sum_x sqrt(Gx(x,y)^2 + Gy(x,y)^2)
```

`Gx` và `Gy` là gradient theo hai hướng. Edge energy càng nhỏ thì ảnh càng mượt và ít cạnh; edge energy càng lớn thì ảnh còn nhiều đường biên hoặc texture hơn.

### Số liệu hiện tại

| Ảnh | Bộ lọc | MSE so với gốc | Edge energy |
|---|---|---:|---:|
| Landscape-fabric-under-gravel | Mean 5x5 | 0.008186 | 0.298188 |
| Landscape-fabric-under-gravel | Gaussian 5x5 | 0.003777 | 0.376200 |
| still_life | Mean 5x5 | 0.000299 | 0.045464 |
| still_life | Gaussian 5x5 | 0.000136 | 0.056112 |
| urban | Mean 5x5 | 0.004230 | 0.159224 |
| urban | Gaussian 5x5 | 0.002130 | 0.198163 |
| urban | Custom | 0.002125 | 0.216446 |

Với cả ba ảnh, Mean có MSE lớn hơn và edge energy nhỏ hơn Gaussian. Điều này cho thấy Mean làm ảnh thay đổi mạnh hơn và làm suy giảm biên nhiều hơn trong cấu hình 5x5 này. Gaussian cho trọng số lớn hơn ở vùng trung tâm nên giữ chi tiết gần vị trí đang xét tốt hơn, vì vậy edge energy cao hơn. Kết luận này đúng cho kernel và sigma đang dùng; nếu đổi kích thước kernel hoặc sigma thì cần chạy lại `metrics.csv`.

Ảnh tĩnh vật có tất cả các giá trị thấp vì ảnh có nhiều vùng phẳng. Ảnh lá/sỏi có các giá trị cao hơn vì chứa nhiều texture. Bộ lọc custom của ảnh `urban` có edge energy cao hơn Gaussian một chút, nên giữ lại nhiều biên hơn, dù MSE gần tương đương.

## 5. Bộ lọc custom tự thiết kế

Bộ lọc này áp dụng riêng lên `urban.png`. Kernel 3x3 lấy mẫu từ tám pixel xung quanh và bỏ qua pixel trung tâm:

```text
1/12 * [1 2 1
        2 0 2
        1 2 1]
```

Kernel lấy mẫu từ tám pixel xung quanh và bỏ giá trị pixel trung tâm. Bốn pixel ngang/dọc có trọng số 2, bốn pixel chéo có trọng số 1; tổng trọng số bằng 12 nên độ sáng ở vùng phẳng gần như được giữ. Bỏ tâm làm thay đổi kết quả rõ hơn và làm mờ chi tiết nhỏ. Đây là kernel tự chọn cho bài này, không phải Mean hay Gaussian. Kết quả lưu ở `urban_custom.png`, ảnh đối chiếu với đầu vào ở `urban_custom_comparison.png`. Nếu tên ảnh custom thay đổi, sửa `CUSTOM_IMAGE_NAME` trong ô cấu hình của notebook.

## 6. High-pass filter

High-pass filter đáp ứng mạnh ở vị trí cường độ sáng thay đổi nhanh. Những vị trí này thường là biên vật thể, đường nét hoặc texture.

Ảnh được chuyển sang grayscale trước khi áp dụng high-pass:

```python
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
```

### Laplacian

```python
laplacian = cv2.Laplacian(gray, cv2.CV_64F, ksize=3)
laplacian = cv2.convertScaleAbs(laplacian)
```

Một kernel Laplacian thường gặp là:

```text
[ 0 -1  0
 -1  4 -1
  0 -1  0]
```

Tổng hệ số bằng 0. Trong vùng có cường độ gần như không đổi, các giá trị triệt tiêu nhau nên kết quả gần 0 và hiển thị tối. Tại đường biên, sự khác biệt cường độ tạo ra giá trị lớn nên biên hiện sáng.

`CV_64F` được dùng vì đạo hàm có thể tạo giá trị âm. `convertScaleAbs` lấy trị tuyệt đối và chuyển về ảnh 8 bit để lưu và hiển thị.

### Sobel

```python
sobel_x = cv2.Sobel(gray, cv2.CV_64F, 1, 0, ksize=3)
sobel_y = cv2.Sobel(gray, cv2.CV_64F, 0, 1, ksize=3)
sobel = cv2.magnitude(sobel_x.astype("float32"), sobel_y.astype("float32"))
```

Hai kernel Sobel cơ bản:

```text
Sobel X = [-1 0 1       Sobel Y = [-1 -2 -1
           -2 0 2                   0  0  0
           -1 0 1]                  1  2  1]
```

- Sobel X đo thay đổi theo phương ngang và làm nổi các cạnh gần thẳng đứng.
- Sobel Y đo thay đổi theo phương dọc và làm nổi các cạnh gần nằm ngang.
- `cv2.magnitude` kết hợp hai kết quả theo công thức `sqrt(Gx^2 + Gy^2)` để nhận được biên ở mọi hướng.

## 7. Nhận xét kết quả để viết báo cáo

### Ảnh đường phố

- Mean và Gaussian làm mờ các cửa sổ, tán cây, xe và vạch đường.
- Laplacian làm nổi nhiều chi tiết nhỏ nhưng các đường biên khá mảnh.
- Sobel tạo biên rõ và liên tục tại thân cây, tòa nhà và vạch kẻ đường.
- Vì ảnh chứa nhiều đường nét nên ảnh high-pass có nhiều vùng sáng.

### Ảnh tĩnh vật

- Bức tường là vùng gần đồng nhất nên ít thay đổi sau low-pass.
- Đường bao quanh cốc, quai cốc và quả táo bị mềm đi sau khi làm trơn.
- Laplacian và Sobel làm nổi chủ yếu đường bao của hai vật thể.
- Phần lớn nền vẫn tối trong ảnh high-pass vì nền không có nhiều thay đổi cường độ.

### Ảnh lá và sỏi

- Mean và Gaussian làm giảm các gân lá và texture nhỏ trên sỏi.
- Laplacian phản ứng mạnh với các chi tiết mảnh và hạt nhỏ.
- Sobel phát hiện được đường bao của lá, sỏi và gân lá nên bản đồ biên dày hơn ảnh tĩnh vật.
- Kết quả cho thấy high-pass không chỉ phát hiện biên vật thể lớn mà còn phản ứng với texture và nhiễu nhỏ.

## 8. So sánh chung

| Bộ lọc | Nhóm | Tác dụng chính | Hạn chế |
|---|---|---|---|
| Mean 5x5 | Low-pass | Làm trơn đơn giản | Làm mờ cạnh khá rõ |
| Gaussian 5x5 | Low-pass | Làm trơn tự nhiên, giảm nhiễu | Có thể làm mất chi tiết nhỏ |
| Laplacian | High-pass | Làm nổi cạnh và chi tiết mảnh | Nhạy với nhiễu |
| Sobel | High-pass | Phát hiện cạnh theo X và Y | Cạnh có thể dày, không phân biệt cạnh thật với nhiễu |

Low-pass và high-pass có tác dụng gần như đối lập. Low-pass phù hợp khi cần giảm nhiễu hoặc làm mượt ảnh trước một bước xử lý khác. High-pass phù hợp khi cần phát hiện biên, phân tích hình dạng hoặc làm nổi chi tiết.

## 9. Nội dung nên đưa vào báo cáo

1. Mục tiêu của phần lọc ảnh.
2. Ba ảnh đầu vào và lý do chọn chúng.
3. Công thức hoặc kernel Mean, Laplacian, Sobel và kernel custom.
4. Giải thích Gaussian filter.
5. Ảnh tổng hợp `<ten>_comparison.png` của từng trường hợp.
6. Nhận xét riêng cho từng ảnh và bảng so sánh chung.
7. Hạn chế: low-pass làm mất chi tiết; high-pass nhạy với nhiễu.

Trong phần code của báo cáo, chỉ cần trích các lệnh từ chuyển grayscale đến Sobel. Phần vòng lặp, kiểm tra file và vẽ hình có thể giữ trong repository mà không cần đưa hết vào báo cáo.

## 10. Prompt để nhờ AI khác chèn nội dung vào LaTeX

Có thể dùng nguyên prompt dưới đây. Khi dùng với bộ ảnh khác, thay các số trong bảng bằng số mới trong `metrics.csv`, không tự bịa số liệu.

```text
Bạn hãy bổ sung vào báo cáo LaTeX hiện tại một subsection cho Phần 2.2 về đánh giá định lượng bộ lọc ảnh. Giữ nguyên cấu trúc, giọng văn và các nội dung đã có; chỉ thêm nội dung cần thiết.

1. Thêm subsection “Metric đánh giá ảnh sau lọc”. Giải thích rằng ảnh được chuyển sang grayscale và chuẩn hóa pixel về [0,1]. Đưa công thức MSE:
MSE(I,F) = 1/(H W) * sum_{y=1}^H sum_{x=1}^W (I(x,y)-F(x,y))^2.
Giải thích MSE đo mức thay đổi so với ảnh gốc, MSE lớn nghĩa là bộ lọc làm thay đổi ảnh nhiều hơn, nhưng MSE không phải điểm chất lượng tuyệt đối vì thí nghiệm không có ảnh chuẩn không nhiễu.

2. Đưa công thức edge energy dựa trên gradient Sobel:
E(F) = 1/(H W) * sum_{x,y} sqrt(G_x(x,y)^2 + G_y(x,y)^2).
Giải thích edge energy thấp thường biểu thị ảnh mượt hơn và mất nhiều cạnh hơn; edge energy cao biểu thị còn nhiều biên hoặc texture hơn.

3. Tạo một bảng so sánh Mean 5x5 và Gaussian 5x5 cho ba ảnh, gồm các cột: tên ảnh, MSE so với ảnh gốc và edge energy. Dùng đúng các số sau:
- Landscape-fabric-under-gravel: Mean MSE 0.008186, edge energy 0.298188; Gaussian MSE 0.003777, edge energy 0.376200.
- still_life: Mean MSE 0.000299, edge energy 0.045464; Gaussian MSE 0.000136, edge energy 0.056112.
- urban: Mean MSE 0.004230, edge energy 0.159224; Gaussian MSE 0.002130, edge energy 0.198163.

4. Viết nhận xét: trong cấu hình 5x5 này, Mean có MSE cao hơn và edge energy thấp hơn Gaussian trên cả ba ảnh, nên Mean làm thay đổi ảnh mạnh hơn và làm yếu biên nhiều hơn. Gaussian ưu tiên pixel gần tâm nên giữ chi tiết tốt hơn. Nêu rõ kết luận phụ thuộc kích thước kernel và sigma.

5. Thêm subsection “Bộ lọc custom tự thiết kế”. Mô tả kernel 3x3 tự chọn:
1/12 * [[1,2,1],[2,0,2],[1,2,1]].
Giải thích kernel bỏ pixel trung tâm, dùng tám pixel xung quanh, trong đó bốn pixel ngang/dọc có trọng số 2 và bốn pixel chéo có trọng số 1. Tổng trọng số bằng 1. Kernel được áp dụng riêng lên ảnh urban.png và kết quả lưu ở urban_custom.png.

6. Thêm số liệu custom của ảnh urban: MSE 0.002125, edge energy 0.216446. Nhận xét rằng custom có edge energy cao hơn Gaussian một chút, nên giữ lại nhiều biên hơn; đây là kernel tự thiết kế cho bài thực nghiệm, không gọi nó là Mean hoặc Gaussian.

Dùng LaTeX chuẩn, đặt công thức trong môi trường equation, bảng trong table/tabular, không tạo số liệu mới và không xóa nội dung cũ.
```
