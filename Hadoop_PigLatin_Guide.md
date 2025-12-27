# 🐷 Hướng Dẫn Cài Đặt và Thực Hành Apache Pig (Pig Latin) Trên Ubuntu

> **Tác giả:** Trần Huy Vũ
> **Phiên bản:** 1.0  
> **Cập nhật:** Tháng 12, 2025  
> **Môi trường:** Ubuntu, Hadoop 3.x, Pig 0.17.0

---

## 📖 Mục Lục

1. [Kiến thức Linux nền tảng](#1-kiến-thức-linux-nền-tảng)
2. [Biến môi trường & nhắc lại Hadoop](#2-biến-môi-trường--nhắc-lại-hadoop)
3. [Cài đặt Apache Pig 0.17.0](#3-cài-đặt-apache-pig-0170)
4. [Chạy Pig: Local mode & MapReduce mode](#4-chạy-pig-local-mode--mapreduce-mode)
5. [Grunt shell – Các lệnh điều khiển](#5-grunt-shell--các-lệnh-điều-khiển-quan-trọng)
6. [Cú pháp Pig Latin cơ bản](#6-cú-pháp-pig-latin-cơ-bản)
7. [Bài thực hành mẫu](#7-bài-thực-hành-mẫu-với-pig)
8. [Lỗi thường gặp và xử lý](#8-lỗi-thường-gặp-và-cách-xử-lý)
9. [Thói quen làm việc an toàn](#9-thói-quen-làm-việc-an-toàn)

---

## 1. Kiến thức Linux nền tảng

Trước khi bắt đầu, hãy nắm vững các lệnh Linux cơ bản để thao tác hệ thống file.

### 1.1 `pwd` – In thư mục hiện tại
Cho biết bạn đang đứng ở đâu trong hệ thống file.

pwd

Kết quả ví dụ: /home/vutran/pig-0.17.0
text

### 1.2 `ls` – Liệt kê file/thư mục
ls # liệt kê đơn giản
ls -l # liệt kê chi tiết (quyền, dung lượng, ngày sửa)
ls -a # hiện cả file ẩn (bắt đầu bằng dấu chấm .)
ls -la # kết hợp chi tiết + file ẩn

text

### 1.3 `cd` – Di chuyển thư mục
cd ten_thu_muc # vào thư mục
cd .. # lên 1 cấp
cd ~ # về thư mục home (quan trọng)
cd / # về thư mục gốc

text

### 1.4 `rm` – Xóa file/thư mục
> ⚠️ **Cảnh báo:** Lệnh `rm -rf` xóa vĩnh viễn không thể phục hồi. Luôn kiểm tra kỹ bằng `pwd` và `ls` trước khi Enter.

rm file.txt # xóa 1 file
rm -r thu_muc # xóa thư mục (hệ thống sẽ hỏi)
rm -rf thu_muc # xóa thư mục (ép buộc, không hỏi lại)

text

### 1.5 `cat`, `nano`, `less` – Xem & Chỉnh sửa
cat file.txt # in nội dung file ra màn hình
nano file.txt # mở trình soạn thảo để sửa file
less file.log # xem file dài, cuộn lên/xuống (nhấn q để thoát)

text

---

## 2. Biến môi trường & nhắc lại Hadoop

### 2.1 Các biến môi trường chính
*   **`JAVA_HOME`**: Thư mục cài đặt Java.
*   **`HADOOP_HOME`**: Thư mục cài đặt Hadoop (VD: `/home/vutran/hadoop-3.4.2`).
*   **`PIG_HOME`**: Thư mục cài đặt Pig (sẽ thiết lập bên dưới).
*   **`PATH`**: Danh sách đường dẫn chứa các file thực thi.

### 2.2 Kiểm tra nhanh
echo $HADOOP_HOME
echo $JAVA_HOME
which hadoop

text
Nếu `which hadoop` trả về đường dẫn, nghĩa là Hadoop đã sẵn sàng.

---

## 3. Cài đặt Apache Pig 0.17.0

### 3.1 Tải và giải nén
cd ~
wget https://downloads.apache.org/pig/pig-0.17.0/pig-0.17.0.tar.gz
tar -xzf pig-0.17.0.tar.gz
mv pig-0.17.0 pig-0.17.0 # (Bước này tùy chọn nếu tên đã đúng)

text

### 3.2 Thiết lập biến môi trường
Mở file cấu hình shell (`.bashrc`):
nano ~/.bashrc

text

Thêm nội dung sau vào cuối file:
export PIG_HOME=/home/vutran/pig-0.17.0
export PATH=$PATH:$PIG_HOME/bin

text

Lưu (`Ctrl+O` -> `Enter`), thoát (`Ctrl+X`), và nạp lại cấu hình:
source ~/.bashrc
pig -version

text
✅ *Nếu hiển thị phiên bản `Apache Pig version 0.17.0` là thành công.*

---

## 4. Chạy Pig: Local mode & MapReduce mode

### 4.1 Local mode (Không dùng HDFS)
Dùng để test logic nhanh với file nằm trên máy local, không cần bật Hadoop.
pig -x local

text
Màn hình sẽ hiện dấu nhắc lệnh: `grunt>`

### 4.2 MapReduce mode (Dùng Hadoop & HDFS)
Yêu cầu Hadoop (NameNode, DataNode) phải đang chạy.

**B1: Khởi động Hadoop (nếu chưa chạy)**
cd $HADOOP_HOME
sbin/start-dfs.sh
jps # Kiểm tra xem có NameNode, DataNode chưa

text

**B2: Chạy Pig**
pig

text
*(Mặc định không có tham số `-x`, Pig sẽ hiểu là dùng mode mapreduce)*.

---

## 5. Grunt shell – Các lệnh điều khiển quan trọng

Khi đang ở dấu nhắc `grunt>`, bạn có thể dùng các lệnh sau:

### 5.1 Vào và thoát
*   **Vào:** `pig` hoặc `pig -x local`
*   **Thoát:**
    ```
    grunt> quit
    ```
    *(Hoặc nhấn `Ctrl+D`)*

### 5.2 Chạy script từ file (`.pig`)
Giả sử có file `script.pig`.

**Cách 1: Chạy bên trong Grunt**
grunt> run script.pig

text
*(Chạy xong vẫn ở lại shell để debug tiếp)*

**Cách 2: Chạy từ Terminal**
pig script.pig

text
*(Chạy xong tự thoát)*

### 5.3 Các lệnh tiện ích
| Lệnh | Mô tả | Ví dụ |
| :--- | :--- | :--- |
| `help` | Liệt kê trợ giúp | `grunt> help` |
| `history` | Xem lịch sử lệnh | `grunt> history` |
| `clear` | Xóa màn hình | `grunt> clear` |
| `set` | Xem/sửa tham số | `grunt> set default_parallel 4` |
| `describe` | Xem schema (cấu trúc) | `grunt> describe students;` |
| `explain` | Xem kế hoạch thực thi | `grunt> explain students;` |
| `illustrate` | Minh họa dòng dữ liệu | `grunt> illustrate students;` |
| `rmf` | Xóa file trên HDFS | `grunt> rmf hdfs:/user/output` |
| `sh` | Chạy lệnh Shell Linux | `grunt> sh ls -la` |

---

## 6. Cú pháp Pig Latin cơ bản

### 6.1 LOAD – Đọc dữ liệu
students = LOAD 'students.txt'
USING PigStorage(',')
AS (id:int, name:chararray, score:double);

text
*   `students`: Tên relation (bảng tạm).
*   `PigStorage(',')`: Chỉ định dấu phân cách là dấu phẩy.

### 6.2 FOREACH … GENERATE – Chọn cột
names = FOREACH students GENERATE name, score;

text
*(Tương đương `SELECT name, score` trong SQL)*

### 6.3 FILTER – Lọc dữ liệu
good = FILTER students BY score >= 8.0;

text
*(Tương đương `WHERE` trong SQL)*

### 6.4 GROUP – Nhóm dữ liệu
by_all = GROUP students ALL;
avg = FOREACH by_all GENERATE AVG(students.score);

text

### 6.5 ORDER & LIMIT
sorted = ORDER students BY score DESC;
top2 = LIMIT sorted 2;

text

### 6.6 JOIN – Nối bảng
joined = JOIN students BY id, classes BY id;

text

---

## 7. Bài thực hành mẫu với Pig

### 7.1 Bài 1 – Local mode: Lọc sinh viên điểm cao
**B1: Tạo dữ liệu mẫu**
cd ~
cat > students.txt << 'EOF'
1,An,8.0
2,Binh,7.5
3,Chi,9.0
4,Duong,6.0
EOF

text

**B2: Chạy Pig local và xử lý**
pig -x local

text
Trong `grunt>`:
students = LOAD 'students.txt' USING PigStorage(',') AS (id:int, name:chararray, score:double);
good = FILTER students BY score >= 8.0;
result = FOREACH good GENERATE name, score;
DUMP result;

text

### 7.2 Bài 2 – MapReduce mode: Tính điểm trung bình
**B1: Đẩy file lên HDFS**
hdfs dfs -mkdir -p /user/vutran/pigdata
hdfs dfs -put ~/students.txt /user/vutran/pigdata/

text

**B2: Viết script `avg_score.pig`**
students = LOAD 'hdfs:/user/vutran/pigdata/students.txt'
USING PigStorage(',')
AS (id:int, name:chararray, score:double);

grouped = GROUP students ALL;
avg_score = FOREACH grouped GENERATE AVG(students.score);

DUMP avg_score;

text

**B3: Chạy script**
pig avg_score.pig

text

### 7.3 Bài 3 – JOIN 2 bảng
**B1: Tạo và upload file lớp học**
cat > classes.txt << 'EOF'
1,Big Data
2,AI
3,Web
4,Mobile
EOF
hdfs dfs -put classes.txt /user/vutran/pigdata/

text

**B2: Viết script `join_students_classes.pig`**
students = LOAD 'hdfs:/user/vutran/pigdata/students.txt' USING PigStorage(',') AS (id:int, name:chararray, score:double);
classes = LOAD 'hdfs:/user/vutran/pigdata/classes.txt' USING PigStorage(',') AS (id:int, classname:chararray);

joined = JOIN students BY id, classes BY id;

result = FOREACH joined GENERATE
students::name,
classes::classname,
students::score;

DUMP result;

text

**B3: Chạy**
pig join_students_classes.pig

text

---

## 8. Lỗi thường gặp và cách xử lý

### 8.1 `pig: command not found`
*   **Nguyên nhân:** Chưa cấu hình biến môi trường.
*   **Khắc phục:** Kiểm tra lại file `~/.bashrc` xem đã có `export PATH=$PATH:$PIG_HOME/bin` chưa, sau đó chạy `source ~/.bashrc`.

### 8.2 Lỗi kết nối HDFS / Connection refused
*   **Nguyên nhân:** Đang chạy Pig mode mặc định nhưng Hadoop chưa bật.
*   **Khắc phục:** Start Hadoop (`start-dfs.sh`) và kiểm tra bằng `jps`.

### 8.3 Input path does not exist
*   **Nguyên nhân:** File chưa được upload lên HDFS hoặc sai đường dẫn.
*   **Khắc phục:** Kiểm tra file trên HDFS bằng lệnh:
    ```
    hdfs dfs -ls /user/vutran/pigdata
    ```

### 8.4 Lỗi Schema / Cast type
*   **Nguyên nhân:** Tính toán số học trên cột kiểu `chararray` hoặc dữ liệu có dòng tiêu đề (header).
*   **Khắc phục:**
    *   Dùng `describe alias;` để xem kiểu dữ liệu.
    *   Dùng `FILTER` để loại bỏ dòng header nếu có.

---

## 9. Thói quen làm việc an toàn
1.  👀 **Nhìn trước khi xóa:** Luôn gõ `pwd` và `ls` để xác nhận vị trí trước khi dùng `rm -rf`.
2.  💾 **Backup:** Sao lưu các script `.pig` và file cấu hình quan trọng.
3.  📝 **Ghi chú:** Tạo file `notes.md` ghi lại các lệnh hay quên.
4.  🐞 **Debug:** Khi gặp lỗi, dùng `describe` và `illustrate` trong Grunt shell để hiểu luồng dữ liệu đang chạy sai ở đâu.
