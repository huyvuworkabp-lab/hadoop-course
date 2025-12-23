# Hướng Dẫn Cài Đặt và Sử Dụng Hadoop & Hive 3.1.3 Trên Ubuntu

**Tác giả:** Trần Huy Vũ
**Phiên bản:** 1.0  
**Ngày cập nhật:** 23 Tháng 12, 2025  

---

## Mục Lục

1. [Kiến Thức Nền Tảng Linux](#kiến-thức-nền-tảng-linux)
2. [Biến Môi Trường và Đường Dẫn](#biến-môi-trường-và-đường-dẫn)
3. [Cài Đặt Hadoop 3.4.2](#cài-đặt-hadoop-342)
4. [Cài Đặt Hive 3.1.3](#cài-đặt-hive-313)
5. [Khởi Tạo và Kiểm Tra Metastore](#khởi-tạo-và-kiểm-tra-metastore)
6. [Cách Đọc Thông Báo Lỗi](#cách-đọc-thông-báo-lỗi)
7. [HDFS vs. Hệ Thống Tệp Linux](#hdfs-vs-hệ-thống-tệp-linux)
8. [Code HiveQL Mẫu](#code-hiveql-mẫu)
9. [Thói Quen An Toàn Khi Thử Nghiệm](#thói-quen-an-toàn-khi-thử-nghiệm)
10. [Các Lỗi Thường Gặp và Cách Giải](#các-lỗi-thường-gặp-và-cách-giải)

---

## Kiến Thức Nền Tảng Linux

Để làm việc với Hadoop và Hive, bạn cần nắm vững một số lệnh Linux cơ bản. Dưới đây là những lệnh **không thể bỏ qua**.

### 1. `pwd` - In Thư Mục Hiện Tại

**Ý nghĩa:** Print Working Directory – cho biết bạn đang ở thư mục nào trong cây thư mục.

```bash
pwd
```

**Kết quả ví dụ:**
```
/home/vutran/hive-3.1.3
```

**Tại sao quan trọng?** Khi bạn chạy lệnh tương đối (như `./bin/hive` hay tìm file `metastore_db`), hệ thống sẽ tìm chúng **từ thư mục hiện tại**. Nếu không biết mình ở đâu, rất dễ gặp lỗi "file not found".

**Thói quen:** Trước khi chạy bất kỳ lệnh nào quan trọng, gõ `pwd` để xác nhận vị trí hiện tại.

---

### 2. `ls` - Liệt Kê File và Thư Mục

**Ý nghĩa:** List – hiển thị danh sách file/thư mục trong thư mục hiện tại.

```bash
ls                    # liệt kê thông thường
ls -l                 # liệt kê chi tiết (kích thước, quyền, ngày sửa)
ls -a                 # liệt kê tất cả (bao gồm file ẩn bắt đầu với .)
ls -la                # kết hợp -l và -a
```

**Ví dụ kết quả:**
```
total 52
drwxr-xr-x 10 vutran vutran  4096 Dec 23 17:24 .
drwxr-xr-x  5 root   root    4096 Dec 15 10:00 ..
drwxr-xr-x  3 vutran vutran  4096 Dec 20 09:15 bin
drwxr-xr-x  2 vutran vutran  4096 Dec 20 09:15 conf
drwxr-xr-x  2 vutran vutran  4096 Dec 23 17:25 metastore_db
drwxr-xr-x  6 vutran vutran  4096 Dec 20 09:15 lib
```

**Tại sao quan trọng?** Trước khi sửa một file, bạn cần biết nó có tồn tại không. Trước khi xóa, bạn cần xem lại danh sách để tránh xóa nhầm.

**Thói quen:** Khi nhìn thấy "file not found" hoặc khi không chắc đã giải nén đúng, gõ `ls` hoặc `ls -la` để kiểm tra.

---

### 3. `cd` - Thay Đổi Thư Mục

**Ý nghĩa:** Change Directory – di chuyển đến thư mục khác.

```bash
cd thư_mục                # chuyển đến thư_mục cụ thể
cd ..                     # lên thư mục cha (up one level)
cd ~                      # về thư mục home của user hiện tại
cd /                      # về thư mục gốc của hệ thống
```

**Ví dụ:**
```bash
cd /home/vutran/hive-3.1.3
cd ..                          # lúc này ở /home/vutran
cd ~                           # lúc này ở /home/vutran
cd /home/vutran/hadoop-3.4.2
```

**Tại sao quan trọng?** Hầu hết công việc trong hướng dẫn này yêu cầu bạn chuyển đến thư mục của Hive hoặc Hadoop rồi mới chạy lệnh.

---

### 4. `rm` và `rm -rf` - Xóa File/Thư Mục

**Ý nghĩa:** Remove – xóa file hoặc thư mục.

```bash
rm tên_file              # xóa file đơn lẻ
rm -r thư_mục           # xóa thư mục và toàn bộ nội dung (hỏi lại)
rm -rf thư_mục          # xóa thư mục và toàn bộ nội dung (không hỏi lại)
```

**⚠️ CẢNH BÁO QUAN TRỌNG:**
- `rm` **không thể phục hồi được**. Khi xóa, file sẽ mất vĩnh viễn.
- `rm -rf` **xóa không hỏi lại** – rất dễ xóa nhầm nếu bạn không cẩn thận.

**Qui tắc an toàn:**
1. Trước khi gõ `rm -rf`, luôn kiểm tra `pwd` để chắc mình đang ở thư mục đúng.
2. Trước khi xóa, luôn chạy `ls` để xem danh sách cần xóa.
3. Nếu xóa nhiều file, xóa từng cái một thay vì xóa toàn bộ cùng lúc.

**Ví dụ an toàn:**
```bash
pwd                              # kiểm tra vị trí
# Kết quả: /home/vutran/hive-3.1.3
ls metastore_db                  # xem thư mục muốn xóa
# Kết quả: (danh sách file trong metastore_db)
rm -rf metastore_db              # xóa khi đã chắc chắn
```

---

### 5. `cat` - Xem Nội Dung File

**Ý nghĩa:** Concatenate – in nội dung file ra màn hình (thường dùng cho file nhỏ).

```bash
cat tên_file
```

**Ví dụ:**
```bash
cat hive-site.xml    # xem nội dung file cấu hình Hive
cat ~/.bashrc        # xem nội dung file biến môi trường
```

**Khi nào dùng?** Khi bạn muốn **xem nhanh** nội dung file để kiểm tra cấu hình có đúng không.

---

### 6. `nano` - Chỉnh Sửa File

**Ý nghĩa:** Nano editor – trình chỉnh sửa văn bản đơn giản.

```bash
nano tên_file
```

**Cách dùng:**
- Gõ nội dung hoặc chỉnh sửa.
- Nhấn `Ctrl+O` để lưu (sau đó nhấn `Enter` để xác nhận).
- Nhấn `Ctrl+X` để thoát.

**Ví dụ:**
```bash
nano ~/.bashrc           # chỉnh sửa file biến môi trường
nano hive-site.xml       # chỉnh sửa file cấu hình Hive
```

**Tại sao quan trọng?** Bạn sẽ phải chỉnh sửa file `~/.bashrc` để thiết lập biến môi trường, và file `hive-site.xml` để cấu hình Hive.

---

### 7. `less` - Xem File Dài

**Ý nghĩa:** Xem file dần từng trang (thay vì `cat` hiển thị toàn bộ cùng lúc).

```bash
less tên_file
```

**Điều khiển:**
- Phím lên/xuống hoặc `Page Up/Page Down` để cuộn.
- Gõ `/` rồi từ khóa để tìm kiếm.
- Gõ `q` để thoát.

**Ví dụ:**
```bash
less derby.log      # xem log của Derby metastore
less ~/.hive/hive.log    # xem log của Hive CLI
```

---

## Biến Môi Trường và Đường Dẫn

### Biến Môi Trường là gì?

Biến môi trường là **tên gọi tắt cho đường dẫn dài**. Thay vì gõ `/home/vutran/hive-3.1.3/bin/hive` mỗi lần, bạn có thể thiết lập:

```bash
export HIVE_HOME=/home/vutran/hive-3.1.3
```

Sau đó gõ `$HIVE_HOME/bin/hive` là được.

### Các biến môi trường quan trọng

| Biến | Ý nghĩa | Ví dụ |
|------|---------|-------|
| `JAVA_HOME` | Thư mục cài đặt Java | `/usr/lib/jvm/java-8-openjdk-amd64` |
| `HADOOP_HOME` | Thư mục cài đặt Hadoop | `/home/vutran/hadoop-3.4.2` |
| `HIVE_HOME` | Thư mục cài đặt Hive | `/home/vutran/hive-3.1.3` |
| `PATH` | Danh sách thư mục chứa lệnh | `$PATH:$HIVE_HOME/bin` |

### Cách thiết lập biến môi trường

1. Mở file `~/.bashrc`:
```bash
nano ~/.bashrc
```

2. Cuộn xuống cuối file, thêm:
```bash
export HADOOP_HOME=/home/vutran/hadoop-3.4.2
export HIVE_HOME=/home/vutran/hive-3.1.3
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HIVE_HOME/bin
```

3. Lưu (`Ctrl+O`, `Enter`, `Ctrl+X`).

4. Nạp lại biến:
```bash
source ~/.bashrc
```

5. Kiểm tra:
```bash
echo $HIVE_HOME
```

Nếu hiển thị `/home/vutran/hive-3.1.3` là đúng. Nếu hiển thị trống, xem lại bước 2.

### Lỗi Biến Môi Trường Phổ Biến

**Lỗi 1: `command not found: hive`**
- Nguyên nhân: Biến `PATH` chưa include thư mục `/bin` của Hive.
- Cách sửa: Mở `~/.bashrc`, thêm `$HIVE_HOME/bin` vào `PATH`, chạy `source ~/.bashrc`.

**Lỗi 2: `$HIVE_HOME không tồn tại`**
- Nguyên nhân: Đường dẫn trong `export HIVE_HOME=...` không khớp với vị trí thực tế.
- Cách kiểm tra: `ls /home/vutran/hive-3.1.3` – nếu báo "No such file", xem lại bước giải nén.
- Cách sửa: Chỉnh lại đường dẫn đúng trong `~/.bashrc`.

**Thói quen an toàn:**
1. Trước khi đặt biến môi trường, **luôn kiểm tra đường dẫn thực tế**:
   ```bash
   ls /home/vutran/hive-3.1.3
   ```
2. Sau khi chỉnh `~/.bashrc`, **luôn chạy**:
   ```bash
   source ~/.bashrc
   echo $HIVE_HOME
   which hive
   ```

---

## Cài Đặt Hadoop 3.4.2

### Bước 1: Tải và Giải Nén

```bash
cd ~
wget https://archive.apache.org/dist/hadoop/common/hadoop-3.4.2/hadoop-3.4.2.tar.gz
tar -xzf hadoop-3.4.2.tar.gz
mv hadoop-3.4.2 hadoop-3.4.2   # đổi tên nếu cần
```

### Bước 2: Kiểm Tra Cài Đặt Java

Hadoop yêu cầu Java. Kiểm tra:

```bash
java -version
```

Nếu lỗi "command not found", cài Java:

```bash
sudo apt-get update
sudo apt-get install openjdk-8-jdk
```

### Bước 3: Thiết Lập Biến Môi Trường

Thêm vào `~/.bashrc`:

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=/home/vutran/hadoop-3.4.2
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

Nạp lại:

```bash
source ~/.bashrc
```

### Bước 4: Chỉnh Sửa Cấu Hình HDFS

Mở `$HADOOP_HOME/etc/hadoop/core-site.xml`:

```bash
nano $HADOOP_HOME/etc/hadoop/core-site.xml
```

Tìm `<configuration>` và chèn:

```xml
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
  </property>
```

Mở `$HADOOP_HOME/etc/hadoop/hdfs-site.xml`:

```bash
nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```

Chèn:

```xml
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>

  <property>
    <name>dfs.namenode.name.dir</name>
    <value>/home/vutran/hadoop_data/namenode</value>
  </property>

  <property>
    <name>dfs.datanode.data.dir</name>
    <value>/home/vutran/hadoop_data/datanode</value>
  </property>
```

### Bước 5: Định Dạng NameNode

```bash
hdfs namenode -format
```

Nếu thấy "Successfully formatted", là thành công.

### Bước 6: Khởi Động Hadoop

```bash
cd $HADOOP_HOME
sbin/start-dfs.sh
```

Kiểm tra:

```bash
jps
```

Phải thấy 3 tiến trình: `NameNode`, `DataNode`, `SecondaryNameNode`.

---

## Cài Đặt Hive 3.1.3

### Bước 1: Tải và Giải Nén

```bash
cd ~
wget https://archive.apache.org/dist/hive/hive-3.1.3/apache-hive-3.1.3-bin.tar.gz
tar -xzf apache-hive-3.1.3-bin.tar.gz
mv apache-hive-3.1.3-bin hive-3.1.3
```

### Bước 2: Thiết Lập Biến Môi Trường

Thêm vào `~/.bashrc`:

```bash
export HIVE_HOME=/home/vutran/hive-3.1.3
export PATH=$PATH:$HIVE_HOME/bin
```

Nạp lại:

```bash
source ~/.bashrc
```

### Bước 3: Tạo File Cấu Hình `hive-site.xml`

Điều này **rất quan trọng** và là nơi thường gặp lỗi.

```bash
cd $HIVE_HOME/conf
cat > hive-site.xml << 'EOF'
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>

  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:derby:metastore_db;create=true</value>
  </property>

  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>org.apache.derby.jdbc.EmbeddedDriver</value>
  </property>

  <property>
    <name>datanucleus.autoCreateSchema</name>
    <value>true</value>
  </property>

  <property>
    <name>datanucleus.fixedDatastore</name>
    <value>false</value>
  </property>

  <property>
    <name>datanucleus.schema.autoCreateAll</name>
    <value>true</value>
  </property>

</configuration>
EOF
```

**Chú ý:** Lệnh `cat > ... << 'EOF'` tạo file chính xác mà không bị lỗi ký tự. Điều này khác với cách copy-paste từ một tài liệu có thể gây lỗi.

---

## Khởi Tạo và Kiểm Tra Metastore

### Bước 1: Chuẩn Bị HDFS

```bash
hdfs dfs -mkdir /tmp
hdfs dfs -chmod g+w /tmp

hdfs dfs -mkdir -p /user/hive/warehouse
hdfs dfs -chmod g+w /user/hive/warehouse
```

### Bước 2: Khởi Tạo Schema Metastore

```bash
cd $HIVE_HOME
rm -rf metastore_db          # xóa DB cũ nếu có
schematool -dbType derby -initSchema
```

Nếu thấy dòng **"schemaTool completed successfully"**, là thành công.

### Bước 3: Kiểm Tra Metastore

```bash
schematool -dbType derby -info
```

Lệnh này sẽ in ra thông tin version schema. Nếu không báo lỗi, metastore đã sẵn sàng.

### Bước 4: Mở Hive Shell

```bash
hive
```

Nếu thấy prompt `hive>`, bạn đã vào thành công. Thử lệnh:

```sql
SHOW DATABASES;
```

Nếu thấy danh sách database (bao gồm `default`), mọi thứ đã sửa thành công.

---

## Cách Đọc Thông Báo Lỗi

Khi gặp lỗi, **đừng hoảng sợ**. Lỗi cung cấp **tất cả thông tin bạn cần** để sửa. Đây là cách đọc lỗi hiệu quả:

### Bước 1: Tìm Dòng Lỗi Chính

Bỏ qua phần "stack trace" dài, tìm **dòng đầu tiên có chữ ERROR, Exception, hoặc FAILED**.

**Ví dụ lỗi 1:**
```
Caused by: com.ctc.wstx.exc.WstxParsingException: Illegal character entity: 
expansion character (code 0x8) at [row,col,system-id]: [3215,96,"file:/home/vutran/hive-3.1.3/conf/hive-site.xml"]
```

**Dòng chính:** "Illegal character entity" + "hive-site.xml"  
**Ý nghĩa:** File `hive-site.xml` có ký tự lạ (không phải UTF-8 hợp lệ).  
**Cách sửa:** Xóa `hive-site.xml` cũ, tạo lại bằng lệnh `cat > ... << 'EOF'` (như bước trên).

**Ví dụ lỗi 2:**
```
java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
```

**Dòng chính:** "Unable to instantiate SessionHiveMetaStoreClient"  
**Ý nghĩa:** Hive không thể kết nối tới metastore (Derby database).  
**Cách sửa:** Kiểm tra:
- Thư mục `metastore_db` có tồn tại không? (`ls $HIVE_HOME/metastore_db`)
- Đã chạy `schematool -initSchema` chưa?
- File `hive-site.xml` có đúng không? (`cat $HIVE_HOME/conf/hive-site.xml`)

**Ví dụ lỗi 3:**
```
java.net.ConnectException: Connection refused
	at java.net.PlainSocketImpl.socketConnect(Native Method)
	at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
	...
```

**Dòng chính:** "Connection refused"  
**Ý nghĩa:** Kết nối bị từ chối (Hadoop/Hive service không chạy).  
**Cách sửa:** Kiểm tra `jps` xem NameNode, DataNode có chạy không. Nếu không, chạy `sbin/start-dfs.sh`.

### Bước 2: Liên Hệ Ngược Lại Với Service

Khi nhìn thấy một từ khóa lỗi, hãy nhớ nó liên quan tới **cái nào**:

| Từ khóa | Service | Cách kiểm tra |
|---------|---------|---------------|
| `WstxParsingException` | Cấu hình XML (Hive/Hadoop) | `cat file.xml` để xem nội dung |
| `SessionHiveMetaStoreClient` | Metastore Hive | `ls $HIVE_HOME/metastore_db` |
| `Connection refused` | Service không chạy | `jps` hoặc `lsof -i :9000` |
| `Permission denied` | Quyền truy cập | `ls -la thư_mục` để xem quyền |
| `No such file or directory` | Đường dẫn sai | `ls đường_dẫn` để kiểm tra |

### Bước 3: Ghi Từ Khóa để Tìm Kiếm

Sau khi xác định lỗi, ghi lại **từ khóa chính** (ví dụ "hive-site.xml parsing error", "metastore derby connection refused") rồi search Google hoặc Stack Overflow.

---

## HDFS vs. Hệ Thống Tệp Linux

Đây là một **nhầm lẫn thường gặp nhất** khi học Hadoop.

### Khác Biệt

| Khía Cạnh | HDFS | Linux File System |
|-----------|------|-------------------|
| **Lệnh** | `hdfs dfs -ls /tmp` | `ls /tmp` |
| **Đường dẫn** | `/tmp` (trên HDFS) | `/tmp` (trên Ubuntu) |
| **Lưu trữ** | Phân tán trên cluster | Cục bộ trên máy |
| **Copy lên** | `hdfs dfs -put file.txt /tmp` | `cp file.txt /tmp` |
| **Xem file** | `hdfs dfs -cat /tmp/file.txt` | `cat /tmp/file.txt` |

### Quy Tắc Vàng

- **Lệnh bắt đầu với `hdfs dfs`** → thao tác trên HDFS.
- **Lệnh thường (ls, cd, rm)** → thao tác trên Linux local filesystem.

### Ví Dụ

```bash
# Tạo thư mục trên HDFS
hdfs dfs -mkdir /user/mydata

# Tạo thư mục trên Linux local
mkdir ~/mydata

# Copy file từ Linux lên HDFS
hdfs dfs -put ~/mydata/data.txt /user/mydata/

# Xem file trên HDFS
hdfs dfs -cat /user/mydata/data.txt

# Xem file trên Linux
cat ~/mydata/data.txt
```

---

## Code HiveQL Mẫu

### Ví Dụ 1: Tạo Database và Table

```sql
-- Vào Hive shell: hive
-- Sau đó gõ các lệnh sau

CREATE DATABASE demo_db;
USE demo_db;

CREATE TABLE employees (
  id   INT,
  name STRING,
  salary DOUBLE
);

SHOW TABLES;
```

### Ví Dụ 2: Insert Dữ Liệu

```sql
INSERT INTO TABLE employees VALUES 
  (1, 'Alice', 50000),
  (2, 'Bob', 60000),
  (3, 'Charlie', 75000);
```

### Ví Dụ 3: Truy Vấn Dữ Liệu

```sql
-- Lấy toàn bộ dữ liệu
SELECT * FROM employees;

-- Tìm nhân viên có lương > 55000
SELECT name, salary FROM employees WHERE salary > 55000;

-- Tính tổng lương
SELECT SUM(salary) as total_salary FROM employees;

-- Tính lương trung bình
SELECT AVG(salary) as avg_salary FROM employees;
```

### Ví Dụ 4: Tạo Table Từ File CSV

```bash
# Tạo file CSV trên Linux
cat > /tmp/sales.csv << 'EOF'
region,amount
North,10000
South,15000
East,12000
West,18000
EOF

# Copy lên HDFS
hdfs dfs -put /tmp/sales.csv /user/hive/warehouse/

# Tạo table trong Hive
hive << 'SQL'
CREATE TABLE sales (
  region STRING,
  amount INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION '/user/hive/warehouse/';

SELECT * FROM sales;
SQL
```

---

## Thói Quen An Toàn Khi Thử Nghiệm

Khi làm việc với Hadoop và Hive, hãy tuân theo những quy tắc này để tránh mất dữ liệu và gây lỗi hệ thống:

### 1. Kiểm Tra Trước Khi Xóa

```bash
# LUÔN làm theo trình tự này:
pwd                                    # kiểm tra vị trí hiện tại
ls                                     # xem danh sách file
ls -la metastore_db                    # xem chi tiết thư mục muốn xóa
rm -rf metastore_db                    # xóa khi đã chắc chắn
```

### 2. Sao Lưu File Cấu Hình

Trước khi chỉnh sửa file cấu hình quan trọng, luôn sao lưu:

```bash
cp hive-site.xml hive-site.xml.bak
cp core-site.xml core-site.xml.bak
cp hdfs-site.xml hdfs-site.xml.bak
```

Nếu sửa sai, có thể quay lại:

```bash
cp hive-site.xml.bak hive-site.xml
```

### 3. Kiểm Tra Service Trước Khi Chạy Lệnh

```bash
jps      # xem các Java processes đang chạy
# Phải thấy: NameNode, DataNode, SecondaryNameNode
```

Nếu không thấy, khởi động HDFS:

```bash
$HADOOP_HOME/sbin/start-dfs.sh
```

### 4. Ghi Chép Các Lệnh Quan Trọng

Tạo file `notes.md` lưu các lệnh hay dùng:

```bash
# notes.md

## Khởi động HDFS
cd $HADOOP_HOME && sbin/start-dfs.sh

## Kiểm tra service
jps

## Vào Hive
hive

## Xem log Hive
tail -n 50 ~/.hive/hive.log

## Tạo lại metastore
cd $HIVE_HOME && rm -rf metastore_db && schematool -dbType derby -initSchema
```

### 5. Tạo Thư Mục Riêng Cho Dự Án

Thay vì xài hệ thống chung, tạo thư mục demo:

```bash
mkdir ~/hive_projects
cd ~/hive_projects
mkdir project_1
cd project_1

# Tất cả công việc cho project này đều ở đây
```

---

## Các Lỗi Thường Gặp và Cách Giải

### Lỗi 1: `command not found: hive`

**Nguyên nhân:** Biến môi trường chưa được thiết lập.

**Cách kiểm tra:**
```bash
echo $HIVE_HOME
which hive
```

**Cách sửa:**
```bash
nano ~/.bashrc
# Thêm: export HIVE_HOME=/home/vutran/hive-3.1.3
# Thêm: export PATH=$PATH:$HIVE_HOME/bin
source ~/.bashrc
```

---

### Lỗi 2: `WstxParsingException: Illegal character entity`

**Nguyên nhân:** File `hive-site.xml` có ký tự lạ.

**Cách sửa:**
```bash
cd $HIVE_HOME/conf
rm hive-site.xml

# Tạo lại bằng lệnh cat (không dùng copy-paste):
cat > hive-site.xml << 'EOF'
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>

  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:derby:metastore_db;create=true</value>
  </property>

  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>org.apache.derby.jdbc.EmbeddedDriver</value>
  </property>

  <property>
    <name>datanucleus.autoCreateSchema</name>
    <value>true</value>
  </property>

  <property>
    <name>datanucleus.fixedDatastore</name>
    <value>false</value>
  </property>

  <property>
    <name>datanucleus.schema.autoCreateAll</name>
    <value>true</value>
  </property>

</configuration>
EOF
```

---

### Lỗi 3: `Unable to instantiate SessionHiveMetaStoreClient`

**Nguyên nhân:** Metastore Derby chưa được khởi tạo.

**Cách sửa:**
```bash
cd $HIVE_HOME
rm -rf metastore_db
schematool -dbType derby -initSchema

# Kiểm tra
schematool -dbType derby -info
```

---

### Lỗi 4: `Connection refused` (NameNode/HDFS)

**Nguyên nhân:** HDFS chưa chạy.

**Cách sửa:**
```bash
cd $HADOOP_HOME
sbin/start-dfs.sh

# Kiểm tra
jps
hdfs dfs -ls /
```

---

### Lỗi 5: `No such file or directory` (khi chạy `cd` hoặc `ls`)

**Nguyên nhân:** Đường dẫn sai hoặc chưa tạo thư mục.

**Cách sửa:**
```bash
# Kiểm tra đường dẫn thực tế
ls ~
ls /home/vutran

# Nếu thư mục không tồn tại, tạo mới
mkdir ~/hive-3.1.3
# Hoặc xem lại đường dẫn trong biến môi trường
echo $HIVE_HOME
```

---

### Lỗi 6: `Permission denied`

**Nguyên nhân:** Không có quyền đọc/ghi file hoặc thư mục.

**Cách sửa:**
```bash
# Xem quyền hiện tại
ls -la hive-site.xml

# Thêm quyền (nếu cần)
chmod 644 hive-site.xml   # quyền đọc-ghi cho owner
chmod 755 metastore_db    # quyền thực thi cho thư mục
```

---

### Lỗi 7: `Derby database lock`

**Nguyên nhân:** Có hai phiên Hive đang chạy cùng lúc, cạnh tranh metastore.

**Cách sửa:**
```bash
# Đóng tất cả phiên Hive
pkill -f hive

# Xóa lock file (nếu có)
cd $HIVE_HOME
rm -rf metastore_db/db.lck

# Khởi động lại
hive
```

---

## Tóm Tắt Quy Trình Cài Đặt Toàn Bộ

1. **Cài Hadoop 3.4.2**
   - Tải, giải nén, thiết lập biến môi trường
   - Chỉnh sửa `core-site.xml` và `hdfs-site.xml`
   - Định dạng NameNode: `hdfs namenode -format`
   - Khởi động: `start-dfs.sh`

2. **Cài Hive 3.1.3**
   - Tải, giải nén, thiết lập biến môi trường
   - Tạo `hive-site.xml` với cấu hình Derby

3. **Chuẩn Bị HDFS**
   - Tạo `/tmp`, `/user/hive/warehouse`
   - Đặt quyền `g+w`

4. **Khởi Tạo Metastore**
   - `schematool -dbType derby -initSchema`
   - Kiểm tra: `schematool -dbType derby -info`

5. **Vào Hive và Test**
   - `hive`
   - `SHOW DATABASES;`
   - `CREATE TABLE ...`

---

## Tài Liệu Tham Khảo

- [Apache Hadoop Documentation](https://hadoop.apache.org/docs/r3.4.2/)
- [Apache Hive Documentation](https://cwiki.apache.org/confluence/display/Hive/Home)
- [Derby Database Documentation](https://db.apache.org/derby/)

---

**Ghi chú:** Tài liệu này được viết dựa trên kinh nghiệm thực tế khi cài đặt Hadoop 3.4.2 + Hive 3.1.3 trên Ubuntu. Nếu gặp vấn đề không có trong tài liệu, hãy tham khảo tài liệu chính thức hoặc tìm kiếm lỗi cụ thể kèm từ khóa "Hadoop 3.4.2" hoặc "Hive 3.1.3" để tìm giải pháp.
