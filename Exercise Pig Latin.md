# Bài Tập Apache Pig (Pig Latin) – Local Mode

**Tác giả:** Trần Huy Vũ
**Phiên bản:** 1.0  
**Ngày cập nhật:** Tháng 12, 2025  

---

## Mục lục

- [Phần 1 – Đề bài](#phần-1--đề-bài)
  - [Nhóm A – Cơ bản (5 bài)](#nhóm-a--cơ-bản-5-bài)
  - [Nhóm B – Trung cấp (5 bài)](#nhóm-b--trung-cấp-5-bài)
  - [Nhóm C – Nâng cao (5 bài)](#nhóm-c--nâng-cao-5-bài)
- [Phần 2 – Bài giải tham khảo](#phần-2--bài-giải-tham-khảo)
  - [Lời giải nhóm A – Cơ bản](#lời-giải-nhóm-a--cơ-bản)
  - [Lời giải nhóm B – Trung cấp](#lời-giải-nhóm-b--trung-cấp)
  - [Lời giải nhóm C – Nâng cao](#lời-giải-nhóm-c--nâng-cao)

---

## Phần 1 – Đề bài

### Yêu cầu chung

- Toàn bộ bài tập chạy ở **Pig Local Mode**:

```bash
pig -x local
```

- Mỗi bài:
1. Tạo file `.txt` bằng lệnh `cat > filename.txt << 'EOF' ... EOF`.
2. Viết script Pig (hoặc gõ trực tiếp trong `grunt>`).
3. Sử dụng **LOAD → FILTER → (FOREACH/ORDER/GROUP/...) → DUMP**.

---

## Nhóm A – Cơ bản (5 bài)

### Bài A1 – Lọc sinh viên xuất sắc

1. Tạo file `students_A1.txt` với nội dung:
```bash
1,Tran Van A,7.5
2,Nguyen Thi B,9.2
3,Le Van C,8.0
4,Pham Thi D,6.5
5,Hoang Van E,8.5
```


2. Định dạng: `id, name, score` (int, string, double).  
3. Yêu cầu:
- LOAD file vào Pig.
- Lọc ra sinh viên có `score >= 8.0`.
- In ra `name` và `score`.

---

### Bài A2 – Nhân viên phòng IT

1. Tạo file `staff_A2.txt`:

```bash
NV01,Nam,Sales,1500
NV02,Binh,IT,2000
NV03,Chi,HR,1800
NV04,Dung,IT,2200
NV05,Hoa,Sales,1600
```


2. Định dạng: `emp_id, name, dept, salary` (string, string, string, int).  
3. Yêu cầu:
- LOAD file.
- Lọc nhân viên có `dept == 'IT'`.
- In ra `emp_id`, `name`, `salary`.

---

### Bài A3 – Log lỗi ERROR

1. Tạo file `log_A3.txt`:

```bash
2025-12-01,08:00,INFO,System started
2025-12-01,08:15,ERROR,Database connection failed
2025-12-01,09:00,WARN,Disk usage 90%
2025-12-01,09:30,ERROR,NullPointerException
2025-12-01,10:00,INFO,User logged out
```


2. Định dạng: `date, time, level, message`.  
3. Yêu cầu:
- LOAD log vào Pig.
- Lọc các dòng có `level == 'ERROR'`.
- In ra `date`, `time`, `message`.

---

### Bài A4 – Sản phẩm giá rẻ còn hàng

1. Tạo file `inventory_A4.txt`:

```bash
SP01,Chuot may tinh,150000,50
SP02,Ban phim co,800000,10
SP03,Lot chuot,50000,100
SP04,Tai nghe,300000,0
SP05,USB 32GB,120000,20
```


2. Định dạng: `pid, pname, price, stock` (string, string, int, int).  
3. Yêu cầu:
- LOAD dữ liệu.
- Lọc sản phẩm thỏa đồng thời:
  - `price < 200000`
  - `stock > 0`
- In ra `pname`, `price`, `stock`.

---

### Bài A5 – Khách hàng VIP theo chi tiêu

1. Tạo file `customers_A5.txt`:

```bash
KH01,Nguyen Van An,5000000
KH02,Le Thi Be,12000000
KH03,Tran Van Binh,800000
KH04,Nguyen Thi Chau,15000000
KH05,Hoang Van An,2000000
```


2. Định dạng: `cid, cname, total_spent` (string, string, int).  
3. Yêu cầu:
- LOAD dữ liệu.
- Lọc khách hàng:
  - có tên chứa chuỗi `"Nguyen"` **HOẶC**
  - `total_spent > 10000000`.
- In ra `cid`, `cname`, `total_spent`.

---

## Nhóm B – Trung cấp (5 bài)

### Bài B1 – Lọc bằng điều kiện phức hợp (AND & OR)

1. File `students_B1.txt`:

```bash
1,An,8.5,IT
2,Binh,7.0,Business
3,Chi,9.0,IT
4,Dung,6.0,Design
5,Em,8.0,Business
```


2. Định dạng: `id, name, score, major`.  
3. Yêu cầu:
- LOAD dữ liệu.
- Lọc sinh viên thỏa **ít nhất một** trong hai điều kiện:
  - `score >= 8.0` **và** `major == 'IT'`
  - `score >= 8.5` (dù học ngành gì).
- In ra toàn bộ thông tin.

---

### Bài B2 – Lọc theo regex tên bắt đầu bằng A hoặc H

1. File `names_B2.txt`:

```bash
1,Anh
2,Ha
3,Binh
4,An
5,Hoa
6,Lan
7,Hoang
```


2. Định dạng: `id, name`.  
3. Yêu cầu:
- LOAD dữ liệu.
- Lọc ra những dòng có `name` bắt đầu bằng chữ `A` **hoặc** `H` (gợi ý: `MATCHES 'A.*'` hoặc `MATCHES 'H.*'`).
- In ra `id`, `name`.

---

### Bài B3 – Sắp xếp top N đơn hàng

1. File `orders_B3.txt`:

```bash
o01,Minh,120000
o02,Lan,450000
o03,Minh,70000
o04,Chi,990000
o05,Lan,200000
o06,Tam,1500000
```

2. Định dạng: `order_id, customer, amount`.  
3. Yêu cầu:
- LOAD dữ liệu.
- Sắp xếp (`ORDER`) các đơn hàng theo `amount` giảm dần.
- Lấy `3` đơn hàng có `amount` cao nhất (`LIMIT`).
- In ra `order_id`, `customer`, `amount`.

---

### Bài B4 – Tính tổng và trung bình điểm theo môn

1. File `scores_B4.txt`:

```bash
An,Math,8.0
An,Physics,7.5
Binh,Math,9.0
Binh,Physics,8.0
Chi,Math,6.5
Chi,Physics,7.0
```

2. Định dạng: `name, subject, score`.  
3. Yêu cầu:
- LOAD dữ liệu.
- Nhóm theo `subject` (mỗi môn một nhóm).
- Với mỗi môn, tính **trung bình điểm** của tất cả sinh viên.
- In ra `subject`, `avg_score`.

---

### Bài B5 – Lọc sản phẩm theo khoảng giá và sắp xếp

1. File `products_B5.txt`:

```bash
P01,Keyboard,250000
P02,Mouse,150000
P03,Monitor,3000000
P04,USB Cable,50000
P05,Laptop,15000000
P06,Headphone,800000
```

2. Định dạng: `pid, pname, price`.  
3. Yêu cầu:
- LOAD dữ liệu.
- Lọc sản phẩm có `price` trong khoảng **[100000, 1000000]**.
- Sắp xếp theo `price` tăng dần.
- In ra `pid`, `pname`, `price`.

---

## Nhóm C – Nâng cao (5 bài)

### Bài C1 – Tính điểm trung bình theo sinh viên

1. File `scores_C1.txt`:

```bash
An,Math,8.0
An,Physics,7.0
An,Chemistry,9.0
Binh,Math,6.0
Binh,Physics,7.0
Chi,Math,9.0
Chi,Physics,8.5
```

2. Định dạng: `name, subject, score`.  
3. Yêu cầu:
- LOAD dữ liệu.
- Nhóm theo `name`.
- Tính trung bình `score` cho từng sinh viên.
- Chỉ giữ lại những sinh viên có **điểm trung bình >= 8.0**, in ra `name`, `avg_score`.

---

### Bài C2 – JOIN danh sách sinh viên và lớp học

1. File `students_C2.txt`:

```bash
1,An,IT
2,Binh,BA
3,Chi,IT
4,Dung,Design
```


2. File `classes_C2.txt`:

```bash
IT,Big Data
BA,Business Analyst
Design,UIUX
```


3. Định dạng:
- `students_C2.txt`: `id, name, major`.
- `classes_C2.txt`: `major, classname`.  
4. Yêu cầu:
- LOAD hai file.
- JOIN theo `major`.
- In ra: `name`, `major`, `classname`.

---

### Bài C3 – Đếm số lỗi mỗi mức độ trong log

1. File `log_C3.txt`:

```bash
2025-12-01,INFO,System started
2025-12-01,ERROR,DB failed
2025-12-01,WARN,Low memory
2025-12-01,ERROR,Null pointer
2025-12-01,INFO,User login
2025-12-01,WARN,Disk 90%
2025-12-01,ERROR,Timeout
```

2. Định dạng: `date, level, message`.  
3. Yêu cầu:
- LOAD dữ liệu.
- Nhóm theo `level`.
- Đếm số dòng (bản ghi) ở mỗi `level` (INFO/WARN/ERROR).
- In ra `level`, `count`.

---

### Bài C4 – Tìm khách hàng chi tiêu cao nhất theo thành phố

1. File `customers_C4.txt`:
```bash
KH01,An,HCMC,5000000
KH02,Binh,Hanoi,12000000
KH03,Chi,HCMC,8000000
KH04,Dung,Hanoi,6000000
KH05,Em,HCMC,15000000
```


2. Định dạng: `cid, cname, city, total_spent`.  
3. Yêu cầu:
- LOAD dữ liệu.
- Với **mỗi `city`**, tìm khách hàng có `total_spent` **cao nhất**.
- In ra `city`, `cid`, `cname`, `total_spent` của khách hàng đó.

*(Gợi ý: có thể dùng GROUP rồi ORDER + LIMIT trong từng nhóm.)*

---

### Bài C5 – Phân tích text đơn giản: đếm số dòng chứa từ khóa

1. File `notes_C5.txt`:
```bash
Pig is a high-level platform for creating MapReduce programs.
Pig Latin is the language used by Pig.
Hive and Pig are often used on top of Hadoop.
This line does not talk about pig.
Another Pig Latin example line.
```

2. Định dạng: chỉ 1 cột `line` (cả dòng).  
3. Yêu cầu:
- LOAD dữ liệu (coi mỗi dòng là một chuỗi).
- Đếm số dòng có chứa từ `"Pig"` (không phân biệt hoa/thường là bonus, còn lại chỉ cần `MATCHES '.*Pig.*'`).
- In ra **tổng số dòng chứa `Pig`**.

---

## Phần 2 – Bài giải tham khảo

### Lời giải nhóm A – Cơ bản

> **Lưu ý:** Dưới đây là script tham khảo, sinh viên có thể viết khác nhưng cùng kết quả.

#### Giải A1 – Lọc sinh viên xuất sắc
```bash
pig -x local
```

```bash
undefined
students = LOAD 'students_A1.txt'
USING PigStorage(',')
AS (id:int, name:chararray, score:double);

good = FILTER students BY score >= 8.0;

result = FOREACH good GENERATE name, score;

DUMP result;
```


---

#### Giải A2 – Nhân viên phòng IT
```bash
staff = LOAD 'staff_A2.txt'
USING PigStorage(',')
AS (emp_id:chararray, name:chararray, dept:chararray, salary:int);

it_staff = FILTER staff BY dept == 'IT';

result = FOREACH it_staff GENERATE emp_id, name, salary;

DUMP result;
```


---

#### Giải A3 – Log lỗi ERROR
```bash
logs = LOAD 'log_A3.txt'
USING PigStorage(',')
AS (date:chararray, time:chararray, level:chararray, message:chararray);

errors = FILTER logs BY level == 'ERROR';

result = FOREACH errors GENERATE date, time, message;

DUMP result;
```


---

#### Giải A4 – Sản phẩm giá rẻ còn hàng
```bash
inv = LOAD 'inventory_A4.txt'
USING PigStorage(',')
AS (pid:chararray, pname:chararray, price:int, stock:int);

cheap_available = FILTER inv BY price < 200000 AND stock > 0;

result = FOREACH cheap_available GENERATE pname, price, stock;

DUMP result;
```


---

#### Giải A5 – Khách hàng VIP theo chi tiêu
```bash
customers = LOAD 'customers_A5.txt'
USING PigStorage(',')
AS (cid:chararray, cname:chararray, total_spent:int);

vip = FILTER customers BY
(cname MATCHES '.Nguyen.') OR (total_spent > 10000000);

result = FOREACH vip GENERATE cid, cname, total_spent;

DUMP result;
```


---

### Lời giải nhóm B – Trung cấp

#### Giải B1 – Điều kiện phức hợp
```bash
students = LOAD 'students_B1.txt'
USING PigStorage(',')
AS (id:int, name:chararray, score:double, major:chararray);

cond1 = FILTER students BY score >= 8.0 AND major == 'IT';
cond2 = FILTER students BY score >= 8.5;

combined = UNION cond1, cond2;

-- Loại trùng (nếu cùng xuất hiện ở cả 2 cond)
distinct_combined = DISTINCT combined;

DUMP distinct_combined;

```

---

#### Giải B2 – Regex tên bắt đầu bằng A hoặc H
```bash
names = LOAD 'names_B2.txt'
USING PigStorage(',')
AS (id:int, name:chararray);

sel = FILTER names BY (name MATCHES 'A.') OR (name MATCHES 'H.');

result = FOREACH sel GENERATE id, name;

DUMP result;
```


---

#### Giải B3 – Top 3 đơn hàng
```bash
orders = LOAD 'orders_B3.txt'
USING PigStorage(',')
AS (order_id:chararray, customer:chararray, amount:int);

sorted = ORDER orders BY amount DESC;

top3 = LIMIT sorted 3;

DUMP top3;
```


---

#### Giải B4 – Trung bình điểm theo môn
```bash
scores = LOAD 'scores_B4.txt'
USING PigStorage(',')
AS (name:chararray, subject:chararray, score:double);

grp = GROUP scores BY subject;

avg_scores = FOREACH grp GENERATE
group AS subject,
AVG(scores.score) AS avg_score;

DUMP avg_scores;
```


---

#### Giải B5 – Lọc theo khoảng giá và sắp xếp
```bash
products = LOAD 'products_B5.txt'
USING PigStorage(',')
AS (pid:chararray, pname:chararray, price:int);

filtered = FILTER products BY price >= 100000 AND price <= 1000000;

sorted = ORDER filtered BY price ASC;

result = FOREACH sorted GENERATE pid, pname, price;

DUMP result;
```


---

### Lời giải nhóm C – Nâng cao

#### Giải C1 – Điểm trung bình theo sinh viên
```bash
scores = LOAD 'scores_C1.txt'
USING PigStorage(',')
AS (name:chararray, subject:chararray, score:double);

grp = GROUP scores BY name;

avg_scores = FOREACH grp GENERATE
group AS name,
AVG(scores.score) AS avg_score;

good = FILTER avg_scores BY avg_score >= 8.0;

DUMP good;
```


---

#### Giải C2 – JOIN sinh viên và lớp học
```bash
students = LOAD 'students_C2.txt'
USING PigStorage(',')
AS (id:int, name:chararray, major:chararray);

classes = LOAD 'classes_C2.txt'
USING PigStorage(',')
AS (major:chararray, classname:chararray);

joined = JOIN students BY major, classes BY major;

result = FOREACH joined GENERATE
students::name AS name,
students::major AS major,
classes::classname AS classname;

DUMP result;
```


---

#### Giải C3 – Đếm số lỗi mỗi mức độ
```bash
logs = LOAD 'log_C3.txt'
USING PigStorage(',')
AS (date:chararray, level:chararray, message:chararray);

grp = GROUP logs BY level;

counts = FOREACH grp GENERATE
group AS level,
COUNT(logs) AS cnt;

DUMP counts;
```


---

#### Giải C4 – Khách hàng chi tiêu cao nhất theo thành phố
```bash
customers = LOAD 'customers_C4.txt'
USING PigStorage(',')
AS (cid:chararray, cname:chararray, city:chararray, total_spent:int);

grp = GROUP customers BY city;

top_per_city = FOREACH grp {
sorted = ORDER customers BY total_spent DESC;
top1 = LIMIT sorted 1;
GENERATE
group AS city,
FLATTEN(top1);
};

DUMP top_per_city;
```

*(Kết quả sẽ có cột: `city, cid, cname, city, total_spent`; có thể dùng `FOREACH` ngoài cùng để chọn lại cột nếu muốn đẹp hơn.)*

---

#### Giải C5 – Đếm số dòng chứa từ "Pig"
```bash
lines = LOAD 'notes_C5.txt'
USING PigStorage('\n')
AS (line:chararray);

has_pig = FILTER lines BY line MATCHES '.Pig.';

grp = GROUP has_pig ALL;

cnt = FOREACH grp GENERATE COUNT(has_pig) AS pig_lines;

DUMP cnt;

```

---
