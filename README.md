# HỆ QUẢN TRỊ CSDL SQL SERVER BÀI 2
## Phần mở đầu:
### Thông tin sinh viên
- Họ và tên: Ngô Văn Nguyên
- Lớp: K59KMT.K01
- MSSV: K235480106052
- Khoa: Điện tử - Ngành: Kĩ thuật máy tính
### Yêu cầu và chọn đề tài 
- Em chọn đề tài: Quản lý Sân Bóng Mini.
- Đề tài này rất trực quan: có sân cỏ, có khách hàng (anh em đi đá bóng) và có phiếu đặt sân. Cấu trúc dữ liệu của nó cực kỳ rõ ràng, giúp bạn dễ dàng ghi điểm phần quan hệ giữa các bảng.
## Nội dung
### Phần 1: Thiết kế và Khởi tạo Cấu trúc Dữ liệu
- Bước 1: Khởi tạo Database
Lệnh này sẽ tạo một "ngôi nhà" riêng cho dữ liệu sân bóng của bạn, đặt tên theo cú pháp  [Tên dự án]_[MaSV]
```sql
CREATE DATABASE [QuanLySanBong_K235480106052];
GO
USE [QuanLySanBong_K235480106052];
GO
```
<img width="960" height="600" alt="image" src="https://github.com/user-attachments/assets/cf51d93a-63d0-48e4-af92-42a9fe083f95" />




- Bước 2: Tạo cấu trúc các Bảng (Tables)
Đây là phần quan trọng nhất, thiết kế DB của bạn.  Khởi tạo 3 bảng: `SanBong`, `KhachHang`, `PhieuDatSan`.
```sql
-- Tạo bảng Sân Bóng
CREATE TABLE [SanBong] (
    [MaSanBong] INT PRIMARY KEY IDENTITY(1,1), -- PK: Khóa chính tự tăng
    [TenSanBong] NVARCHAR(100) NOT NULL,
    [LoaiSan] NVARCHAR(20),
    [GiaThueTheoGio] DECIMAL(18, 2), 
    CONSTRAINT [CK_GiaThue] CHECK ([GiaThueTheoGio] > 0) -- CK: Giá phải lớn hơn 0
);
-- Tạo bảng Khách Hàng
CREATE TABLE [KhachHang] (
    [MaKhachHang] INT PRIMARY KEY IDENTITY(1,1), -- PK: Khóa chính
    [HoTenKhachHang] NVARCHAR(100) NOT NULL,
    [SoDienThoai] VARCHAR(15) UNIQUE,
    [DiemTichLuy] INT DEFAULT 0,
    CONSTRAINT [CK_DiemTichLuy] CHECK ([DiemTichLuy] >= 0) -- CK: Điểm không âm
);
-- Tạo bảng Phiếu Đặt Sân
CREATE TABLE [PhieuDatSan] (
    [MaPhieuDat] INT PRIMARY KEY IDENTITY(1,1),
    [MaSanBong] INT, -- FK: Khóa ngoại
    [MaKhachHang] INT, -- FK: Khóa ngoại
    [NgayDat] DATE DEFAULT GETDATE(),
    [SoGioThue] FLOAT,
    CONSTRAINT [FK_PhieuDat_SanBong] FOREIGN KEY ([MaSanBong]) REFERENCES [SanBong]([MaSanBong]),
    CONSTRAINT [FK_PhieuDat_KhachHang] FOREIGN KEY ([MaKhachHang]) REFERENCES [KhachHang]([MaKhachHang]),
    CONSTRAINT [CK_SoGioThue] CHECK ([SoGioThue] > 0)
);
```

<img width="960" height="600" alt="image" src="https://github.com/user-attachments/assets/59c21b24-9033-4f3b-8a5b-7c40c4059558" />




Bước 3: Chèn dữ liệu mẫu (Insert Data) và truy vấn kết quả
Cần có dữ liệu để kết quả thực thi hơn


```sql
-- Thêm dữ liệu sân
INSERT INTO [SanBong] ([TenSanBong], [LoaiSan], [GiaThueTheoGio])
VALUES (N'Sân Mỹ Đình 1', N'Sân 7', 300000),
       (N'Sân Camp Nou 2', N'Sân 5', 200000);

-- Thêm dữ liệu khách
INSERT INTO [KhachHang] ([HoTenKhachHang], [SoDienThoai], [DiemTichLuy])
VALUES (N'Nguyễn Văn Công', '0912345678', 10),
       (N'Trần Duy Mạnh', '0988888888', 5);

-- Truy vấn để xem kết quả
SELECT * FROM [SanBong];
SELECT * FROM [KhachHang];
```


<img width="960" height="600" alt="image" src="https://github.com/user-attachments/assets/6520e195-b0ac-45b3-86c4-68ae37ddbc5b" />




Bước 4: Kiểm tra ràng buộc (Test Constraints)

```sql
-- Thử thêm một sân bóng có giá thuê bằng -50.000 (Sai ràng buộc CK)
INSERT INTO [SanBong] ([TenSanBong], [LoaiSan], [GiaThueTheoGio])
VALUES (N'Sân Lỗi', N'Sân 5', -50000);
```
<img width="960" height="600" alt="image" src="https://github.com/user-attachments/assets/8c57d911-6a5b-4ff8-9d0c-2f6d5168f50c" />

### Phần 2: Xây dựng Function
#### Tìm hiểu về Built-in Function (Hàm có sẵn)Trong SQL Server, các hàm có sẵn được chia thành nhiều nhóm chính:
Hàm chuỗi: LEN, LEFT, RIGHT, REPLACE, UPPER, LOWER.
Hàm ngày tháng: GETDATE, DATEADD, DATEDIFF, YEAR, MONTH, DAY.
Hàm toán học: ROUND, ABS, RAND, SQRT.
Hàm tổng hợp (Aggregate): SUM, AVG, COUNT, MIN, MAX.
Hệ thống hàm Built-in "đặc sắc"
#### Dưới đây là 2 hàm rất hữu ích mà em chọn
- COALESCE: Trả về giá trị đầu tiên không bị NULL trong danh sách. Cực kỳ hữu dụng khi làm báo cáo để thay thế các khoảng trống dữ liệu bằng giá trị mặc định.
- FORMAT: Giúp định dạng dữ liệu (ngày tháng, tiền tệ) theo ngôn ngữ cụ thể cực nhanh.
```sql 
SELECT 
    [TenSanBong],
    FORMAT([GiaThueTheoGio], 'N0', 'vi-VN') + ' VNĐ' AS GiaDinhDang, -- Định dạng tiền kiểu VN
    COALESCE([LoaiSan], N'Chưa xác định') AS LoaiSanCheck -- Nếu NULL thì hiện 'Chưa xác định'
FROM [SanBong];
```

<img width="959" height="600" alt="image" src="https://github.com/user-attachments/assets/08d79a8a-7b18-4923-b3ec-837df9bcea47" />

###  Tìm hiểu về User-Defined Functions (UDF - Hàm tự định nghĩa)
### Mục đích và Phân loại Hàm (UDF)

Hàm do người dùng tự viết dùng để đóng gói các logic tính toán lặp đi lặp lại, giúp mã SQL gọn gàng và dễ bảo trì hơn.

| Loại hàm | Đặc điểm | Khi nào dùng? |
| :--- | :--- | :--- |
| **Scalar Function** | Trả về duy nhất **1 giá trị** (số, chuỗi, ngày...). | Khi cần tính toán đơn giản (ví dụ: tính thuế, tính tổng tiền 1 dòng). |
| **Inline Table-Valued** | Trả về một **bảng dữ liệu** từ một câu lệnh `SELECT` duy nhất. | Khi cần tạo ra một "View động" có tham số để lọc dữ liệu nhanh. |
| **Multi-statement TVF** | Trả về một bảng nhưng có cấu trúc phức tạp, xử lý qua nhiều bước. | Khi logic lọc dữ liệu rất khó, cần dùng vòng lặp, `IF...ELSE` hoặc bảng tạm. |


### Viết Function cho QuanLySanBong_K235480106052
- Scalar Function: Tính tổng tiền thuê sân
Yêu cầu: Tạo hàm tính tổng tiền dựa trên số giờ thuê và đơn giá sân, có tính thêm 10% VAT.
```sql
GO
CREATE FUNCTION dbo.fn_TinhTongTienCoThue (
    @GiaThue DECIMAL(18,2),
    @SoGio FLOAT
)
RETURNS DECIMAL(18,2)
AS
BEGIN
    RETURN (@GiaThue * @SoGio) * 1.1; -- Tính tổng và cộng 10% VAT
END;
GO

-- KHAI THÁC HÀM:
SELECT 
    [MaPhieuDat],
    dbo.fn_TinhTongTienCoThue(300000, 2.5) AS TongTienCoThue
FROM [PhieuDatSan];
```

<img width="960" height="598" alt="image" src="https://github.com/user-attachments/assets/23b25c22-bd13-480e-ad4d-7fc7c70aff83" />

 
 
 ### BAN ĐẦU ĐOẠN CODE CHẠY BỊ LỖI KHÔNG HIỂN THỊ RA GÌ DO THIẾU THÔNG TIN NÊN EM ĐÃ CHÈN LẠI THÔNG TIN BẰNG ĐOẠN CODE SAU: 

 
 ```Sql
 USE [QuanLySanBong_K235480106052];
GO

-- 1. Thêm lại Sân và Khách (nếu bạn đã lỡ xóa sạch)
INSERT INTO [SanBong] ([TenSanBong], [LoaiSan], [GiaThueTheoGio]) VALUES (N'Sân Wembley', N'Sân 7', 300000);
INSERT INTO [KhachHang] ([HoTenKhachHang]) VALUES (N'Phạm Quang Hải');

-- 2. Thêm dữ liệu vào bảng PhieuDatSan (Để hàm có cái mà tính)
INSERT INTO [PhieuDatSan] ([MaSanBong], [MaKhachHang], [SoGioThue])
VALUES (1, 1, 2.5), (1, 1, 1.5);
GO
```


- Inline Table-Valued Function: Tìm sân theo loại
Yêu cầu: Nhập vào loại sân (Sân 5, Sân 7...), trả về danh sách các sân thuộc loại đó.

```sql
GO
CREATE FUNCTION dbo.fn_DanhSachSanTheoLoai (@LoaiSan NVARCHAR(20))
RETURNS TABLE
AS
RETURN (
    SELECT [MaSanBong], [TenSanBong], [GiaThueTheoGio]
    FROM [SanBong]
    WHERE [LoaiSan] = @LoaiSan
);
GO

-- KHAI THÁC HÀM:
SELECT * FROM dbo.fn_DanhSachSanTheoLoai(N'Sân 7');
```

<img width="960" height="600" alt="image" src="https://github.com/user-attachments/assets/1e61f551-d662-4e79-b1e1-b714f59ed111" />






- Multi-statement Table-Valued Function: Phân hạng khách hàng
Yêu cầu: Dựa vào điểm tích lũy, xuất ra bảng danh sách khách hàng kèm theo cột "Phân Hạng" (VIP nếu điểm > 50, Thân thiết nếu > 20, còn lại là Thành viên).

```sql
GO
CREATE FUNCTION dbo.fn_PhanHangKhachHang()
RETURNS @DanhSachPhanHang TABLE (
    [MaKH] INT,
    [TenKH] NVARCHAR(100),
    [Diem] INT,
    [Hang] NVARCHAR(50)
)
AS
BEGIN
    INSERT INTO @DanhSachPhanHang
    SELECT [MaKhachHang], [HoTenKhachHang], [DiemTichLuy],
           CASE 
                WHEN [DiemTichLuy] >= 50 THEN N'Khách VIP'
                WHEN [DiemTichLuy] >= 20 THEN N'Khách Thân Thiết'
                ELSE N'Thành Viên'
           END
    FROM [KhachHang];
    
    RETURN;
END;
GO

-- KHAI THÁC HÀM:
SELECT * FROM dbo.fn_PhanHangKhachHang();
```
<img width="960" height="600" alt="image" src="https://github.com/user-attachments/assets/a75ad66e-b8d5-4e0a-b33a-2428f350e973" />


### Phần 3: Xây dựng Store Procedure

#### Tìm hiểu về System Store Procedure (SP có sẵn)
Trong SQL Server, các System SP thường bắt đầu bằng tiền tố sp_ và được lưu trong database master. Chúng giúp quản trị viên kiểm tra hệ thống cực nhanh.

Một vài System SP đặc sắc:
- sp_help: Cung cấp thông tin chi tiết về bất kỳ đối tượng nào (bảng, view, index).

Cách dùng: EXEC sp_help 'SanBong'; -> Nó sẽ hiện ra bảng này có những cột nào, kiểu dữ liệu gì, khóa chính là ai.

- sp_helpdb: Liệt kê danh sách các database đang có và dung lượng của chúng.

Cách dùng: EXEC sp_helpdb;

- sp_rename: Dùng để đổi tên một đối tượng (như đổi tên bảng hoặc tên cột) mà không cần xóa đi tạo lại.

Cách dùng: EXEC sp_rename 'TenCu', 'TenMoi';
Thực hành viết Store Procedure
#### SP Thêm mới Sân Bóng (Có kiểm tra logic)
Yêu cầu: Tạo SP để thêm sân mới. Nếu giá thuê nhỏ hơn hoặc bằng 0 thì không cho thêm và báo lỗi.
```sql
GO
CREATE PROCEDURE sp_ThemSanBong (
    @TenSan NVARCHAR(100),
    @LoaiSan NVARCHAR(20),
    @GiaThue DECIMAL(18,2)
)
AS
BEGIN
    -- Kiểm tra logic: Giá thuê phải > 0
    IF (@GiaThue <= 0)
    BEGIN
        PRINT N'Lỗi: Giá thuê sân phải lớn hơn 0!';
    END
    ELSE
    BEGIN
        INSERT INTO [SanBong] ([TenSanBong], [LoaiSan], [GiaThueTheoGio])
        VALUES (@TenSan, @LoaiSan, @GiaThue);
        PRINT N'Thêm sân bóng thành công!';
    END
END;
GO

-- KHAI THÁC (Thử trường hợp đúng):
EXEC sp_ThemSanBong N'Sân Old Trafford', N'Sân 11', 500000;
```

<img width="960" height="600" alt="image" src="https://github.com/user-attachments/assets/f511b268-67c6-442b-83ae-0d9386f498a8" />

#### SP Sử dụng tham số OUTPUT
Yêu cầu: Đếm xem một khách hàng (theo mã) đã đặt sân bao nhiêu lần. Giá trị số lần đặt trả về qua tham số OUTPUT.

```sql
GO
CREATE PROCEDURE sp_DemSoLanDatSan (
    @MaKH INT,
    @SoLanDat INT OUTPUT -- Tham số trả kết quả ra ngoài
)
AS
BEGIN
    SELECT @SoLanDat = COUNT(*) 
    FROM [PhieuDatSan] 
    WHERE [MaKhachHang] = @MaKH;
END;
GO

-- KHAI THÁC:
DECLARE @Result INT;
EXEC sp_DemSoLanDatSan @MaKH = 1, @SoLanDat = @Result OUTPUT;
SELECT N'Số lần khách hàng 1 đã đặt sân là: ' + CAST(@Result AS NVARCHAR);
```

<img width="960" height="600" alt="image" src="https://github.com/user-attachments/assets/6f14774a-3ab5-42fd-96e9-ac8abd6dfa70" />

#### SP Trả về tập kết quả (Result Set) Join nhiều bảng
Yêu cầu: Xuất ra "Hóa đơn chi tiết" bao gồm: Tên khách hàng, Tên sân đã đặt, và Ngày đặt.
```sql
GO
CREATE PROCEDURE sp_LayChiTietDatSan
AS
BEGIN
    SELECT 
        K.[HoTenKhachHang] AS [TenKhach],
        S.[TenSanBong] AS [TenSan],
        P.[NgayDat] AS [NgayDatSan],
        S.[LoaiSan]
    FROM [PhieuDatSan] P
    INNER JOIN [KhachHang] K ON P.[MaKhachHang] = K.[MaKhachHang]
    INNER JOIN [SanBong] S ON P.[MaSanBong] = S.[MaSanBong]
    ORDER BY P.[NgayDat] DESC;
END;
GO

-- KHAI THÁC:
EXEC sp_LayChiTietDatSan;
```

<img width="960" height="600" alt="image" src="https://github.com/user-attachments/assets/617c3d03-7d4e-4be3-81e3-b5f3121e42af" />


### Phần 4: Trigger và Xử lý logic nghiệp vụ

- Viết 01 Trigger xử lý logic thực tế
Kịch bản: Khi một anh em đặt sân bóng mới (INSERT vào bảng PhieuDatSan), hệ thống sẽ tự động cộng 5 điểm thưởng cho khách hàng đó vào bảng KhachHang. Điều này giúp quản lý điểm tích lũy hoàn toàn tự động.

```sql
GO
CREATE TRIGGER trg_CapNhatDiemTichLuy
ON [PhieuDatSan]
AFTER INSERT
AS
BEGIN
    -- Lấy MaKhachHang từ dòng vừa được chèn vào (bảng virtual inserted)
    UPDATE [KhachHang]
    SET [DiemTichLuy] = [DiemTichLuy] + 5
    FROM [KhachHang] K
    JOIN inserted i ON K.[MaKhachHang] = i.[MaKhachHang];
    
    PRINT N'Hệ thống: Đã tự động cộng 5 điểm thưởng cho khách hàng!';
END;
GO

-- KHAI THÁC (Thử đặt 1 sân mới):
INSERT INTO [PhieuDatSan] ([MaSanBong], [MaKhachHang], [SoGioThue])
VALUES (1, 1, 1.5);

-- Kiểm tra điểm khách hàng đã tăng chưa
SELECT * FROM [KhachHang] WHERE [MaKhachHang] = 1;
```

<img width="960" height="600" alt="image" src="https://github.com/user-attachments/assets/cffd8d24-a124-42bd-a880-4d222ad6d045" />

- Thử nghiệm Trigger vòng lặp (Recursion Trigger)
Đây là phần thí nghiệm để bạn quan sát lỗi hệ thống. Chúng ta sẽ tạo ra một tình huống: "A gọi B, B gọi lại A".

**Bước A**: Tạo Trigger trên bảng KhachHang (Cập nhật sang SanBong)
```sql
GO
CREATE TRIGGER trg_KhachHang_To_SanBong
ON [KhachHang]
AFTER UPDATE
AS
BEGIN
    PRINT N'Trigger A đang chạy... cập nhật bảng SanBong';
    UPDATE [SanBong] SET [LoaiSan] = [LoaiSan] WHERE [MaSanBong] = 1;
END;
GO
```
<img width="960" height="600" alt="image" src="https://github.com/user-attachments/assets/59f31bb1-38b2-414d-9878-84209c461b5e" />

**Bước B**: Tạo Trigger trên bảng SanBong (Cập nhật ngược lại KhachHang)
```sql
GO
CREATE TRIGGER trg_SanBong_To_KhachHang
ON [SanBong]
AFTER UPDATE
AS
BEGIN
    PRINT N'Trigger B đang chạy... cập nhật ngược lại bảng KhachHang';
    UPDATE [KhachHang] SET [HoTenKhachHang] = [HoTenKhachHang] WHERE [MaKhachHang] = 1;
END;
GO
```
<img width="960" height="600" alt="image" src="https://github.com/user-attachments/assets/821679d8-3991-4dcb-af43-edf0edf6dc37" />

**Bước C**: Kích hoạt tình trạng lỗi
Chạy lệnh này và quan sát tab Messages:
```sql
UPDATE [KhachHang] SET [DiemTichLuy] = 100 WHERE [MaKhachHang] = 1;
```

<img width="960" height="600" alt="image" src="https://github.com/user-attachments/assets/1f3c02a9-0198-4d01-8438-7c9f61bcea79" />

#### Quan sát và Nhận xét
- Hiện tượng quan sát được:
Khi bạn chạy lệnh Update ở Bước C, hệ thống sẽ hiện ra một loạt thông báo lặp đi lặp lại:

Trigger A đang chạy...

Trigger B đang chạy...

...

Cuối cùng, một thông báo lỗi màu đỏ xuất hiện: "Maximum stored procedure, function, trigger, or view nesting level exceeded (limit 32)."

#### Giải thích thông báo:
- Nesting level exceeded: SQL Server có một cơ chế bảo vệ, nó chỉ cho phép các lệnh gọi nhau tối đa 32 tầng.

- Khi Trigger A kích hoạt Trigger B, và B lại kích hoạt ngược lại A, nó tạo thành một cái "vòng lặp vô tận" (Infinite Loop). Hệ thống phát hiện ra việc này sẽ tự động ngắt lệnh để tránh làm treo máy chủ (tràn bộ nhớ).

#### Nhật xét cuối cùng:
- Tính nguy hiểm: Tình trạng này được gọi là Indirect Recursion (Đệ quy gián tiếp). Nó cực kỳ nguy hiểm vì có thể gây treo hệ thống hoặc làm sai lệch dữ liệu hàng loạt nếu không có giới hạn 32 tầng của SQL Server.

- Kinh nghiệm thiết kế: Khi viết Trigger, tuyệt đối tránh việc cập nhật chéo giữa các bảng theo vòng tròn. Nếu cần cập nhật nhiều bảng, nên sử dụng Store Procedure để kiểm soát luồng dữ liệu thay vì dùng Trigger chồng chéo.
