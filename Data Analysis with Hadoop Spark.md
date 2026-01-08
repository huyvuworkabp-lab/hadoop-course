<div align="center">

# Data Analysis with Hadoop Spark
## Case Study: PHÂN TÍCH DỮ LIỆU SIÊU THỊ

[![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![PySpark](https://img.shields.io/badge/PySpark-3.5+-F8961E?style=for-the-badge&logo=apachespark&logoColor=white)](https://spark.apache.org)
[![Google Colab](https://img.shields.io/badge/Google%20Colab-FEE71C?style=for-the-badge&logo=google-colab&logoColor=white)](https://colab.research.google.com)
</div>

**Dataset**: Supermarket Sales (1000+ giao dịch, 17 cột: Invoice ID, Branch, City, Customer type, Gender, Product line, Unit price, Quantity, Date, Time, Payment, cogs, Total, Tax 5%, Rating...)


**Mục tiêu**: Phân tích dữ liệu bán hàng, **phân cụm khách hàng (K-Means)**, **dự báo doanh thu (Linear Regression)**.



---

## 📋 Mục lục

- [1. Setup Spark](#1-setup-spark)
- [2. Load và Chuẩn bị Dữ liệu](#2-load-và-chuẩn-bị-dữ-liệu)
- [3. Làm sạch Dữ liệu](#3-làm-sạch-dữ-liệu)
- [4. Phân tích Thống kê](#4-phân-tích-thống-kê)
- [5. Phân cụm Khách hàng (K-Means)](#5-phân-cụm-khách-hàng-k-means)
- [6. Dự báo Doanh thu (Linear Regression)](#6-dự-báo-doanh-thu-linear-regression)
- [7. Export Kết quả](#7-export-kết-quả)

---

## 1. Setup Spark

```python
# Cài đặt thư viện PySpark và findspark (chạy 1 lần duy nhất)
# -q: quiet mode, không hiển thị quá nhiều log khi cài đặt
!pip install pyspark findspark -q

# Import findspark để khởi tạo môi trường Spark trong Google Colab
import findspark
findspark.init()  # Tìm và cấu hình đường dẫn Spark trong hệ thống

# Import SparkSession - lớp chính để làm việc với Spark DataFrame và ML
from pyspark.sql import SparkSession

# Import các hàm SQL từ pyspark.sql.functions
# col: truy cập cột DataFrame
# sum as F_sum: tính tổng (đổi tên tránh xung đột với hàm sum() của Python)
# avg as F_avg: tính trung bình
# count as F_count: đếm số dòng
# to_date: chuyển đổi string thành kiểu Date
# row_number: tạo số thứ tự tăng dần cho mỗi dòng
from pyspark.sql.functions import col, sum as F_sum, avg as F_avg, count as F_count, to_date, row_number

# Import các công cụ Machine Learning
from pyspark.ml.feature import VectorAssembler, StandardScaler  # Feature engineering
from pyspark.ml.clustering import KMeans  # Thuật toán phân cụm
from pyspark.ml.evaluation import ClusteringEvaluator  # Đánh giá mô hình clustering
from pyspark.ml.regression import LinearRegression  # Hồi quy tuyến tính
from pyspark.sql.window import Window  # Window functions cho tính toán theo nhóm

# Tạo Spark Session - điểm khởi đầu để làm việc với Spark
# builder: tạo session với pattern Builder
# appName: đặt tên ứng dụng (hiển thị trên Spark UI)
# getOrCreate(): lấy session hiện tại hoặc tạo mới nếu chưa có
spark = SparkSession.builder.appName("SupermarketAnalysis").getOrCreate()
spark  # Hiển thị thông tin Spark Session (URL của Spark UI để debug)
```

---

## 2. Load và Chuẩn bị Dữ liệu

```python
# ID file trên Google Drive (lấy từ link chia sẻ)
file_id = "1Fa7SxYpLDUjx2-lbjMhx1udDJd4ZZOMW"

# Tải file CSV từ Google Drive xuống Colab bằng wget
# f"...": f-string để chèn giá trị file_id vào URL
# -q: quiet mode, không hiển thị progress bar
# -O supermarket.csv: lưu file với tên supermarket.csv
!wget -q f"https://drive.google.com/uc?export=download&id={file_id}" -O supermarket.csv

# Đọc file CSV thành Spark DataFrame
# spark.read: đối tượng đọc dữ liệu
# option("header", "true"): dòng đầu tiên là tên cột
# option("inferSchema", "true"): tự động đoán kiểu dữ liệu (int, string, double...)
# csv("supermarket.csv"): đọc file CSV
df = spark.read.option("header", "true").option("inferSchema", "true").csv("supermarket.csv")

# Tạo dictionary ánh xạ tên cột cũ -> tên cột mới
# Bỏ dấu space trong tên cột để tránh lỗi khi truy cập df."Column Name"
column_mapping = {
    "Invoice ID": "InvoiceID",  # Mã hóa đơn
    "Customer type": "CustomerType",  # Loại khách hàng
    "Product line": "ProductLine",  # Dòng sản phẩm
    "Unit price": "UnitPrice"  # Đơn giá
}

# Đổi tên cột theo mapping
# df.columns: danh sách tất cả tên cột
# column_mapping.get(c, c): lấy tên mới từ mapping, giữ nguyên nếu không có trong mapping
# col(c).alias(...): đổi tên cột c thành tên mới
# df.select([...]): chọn tất cả cột với tên đã đổi
df = df.select([col(c).alias(column_mapping.get(c, c)) for c in df.columns])

# Tạo các cột mới cho phân tích
# withColumn: thêm cột mới hoặc ghi đè cột cũ
# to_date(col("Date"), "M/d/yyyy"): chuyển string "1/5/2019" thành Date object
# - M: tháng không có số 0 đầu (1-12)
# - d: ngày không có số 0 đầu (1-31)
# - yyyy: năm 4 chữ số
df = df.withColumn("RealDate", to_date(col("Date"), "M/d/yyyy")) \
       .withColumn("Revenue", col("Total") - col("Tax5%")) \  # Doanh thu = Tổng tiền - Thuế
       .withColumn("Profit", col("Revenue") - col("cogs"))  # Lợi nhuận = Doanh thu - Giá vốn hàng

# Hiển thị 5 dòng đầu tiên để kiểm tra
df.show(5)

# In cấu trúc DataFrame (tên cột, kiểu dữ liệu, nullable)
df.printSchema()
```

---

## 3. Làm sạch Dữ liệu

```python
# Xóa các dòng có giá trị NULL ở 3 cột quan trọng
# dropna(subset=[...]): chỉ drop nếu NULL ở các cột trong subset
# Giữ lại các dòng NULL ở cột khác (vd: Gender, City...)
df_clean = df.dropna(subset=["UnitPrice", "Quantity", "Rating"])

# Xóa các dòng trùng lặp dựa trên InvoiceID (khóa chính)
# dropDuplicates(["InvoiceID"]): giữ lại dòng đầu tiên, xóa các dòng trùng sau
df_clean = df_clean.dropDuplicates(["InvoiceID"])

# Loại bỏ outlier (giá trị bất thường) của cột UnitPrice
# Phương pháp: 3-sigma rule (giữ giá trị trong khoảng mean ± 3*std)
from pyspark.sql.functions import stddev_pop, avg

# Tính mean và standard deviation của UnitPrice
# agg: aggregation function (tính toán trên toàn bộ DataFrame)
# collect(): chuyển kết quả từ Spark DataFrame về Python list
# Kết quả: [Row(avg=..., stddev_pop=...)]
price_stats = df_clean.agg(avg("UnitPrice"), stddev_pop("UnitPrice")).collect()

# Tính ngưỡng trên: mean + 3 * standard_deviation
# price_stats[0]: dòng đầu tiên (Row object)
# price_stats[0][0]: cột đầu tiên (mean)
# price_stats[0][1]: cột thứ 2 (stddev)
upper_bound = price_stats[0][0] + 3 * price_stats[0][1]

# Lọc chỉ giữ dòng có UnitPrice <= upper_bound
# filter(): giống WHERE trong SQL
df_clean = df_clean.filter(col("UnitPrice") <= upper_bound)

# In số dòng trước và sau khi làm sạch
# count(): đếm tổng số dòng trong DataFrame
print("Original:", df.count(), "Cleaned:", df_clean.count())
```

---

## 4. Phân tích Thống kê

```python
# Tính các chỉ số thống kê theo từng Chi nhánh (Branch)
# groupBy("Branch"): nhóm dữ liệu theo cột Branch (A, B, C)
# agg(): áp dụng nhiều hàm tổng hợp cùng lúc
branch_stats = df_clean.groupBy("Branch").agg(
    # F_sum("Revenue"): tính tổng doanh thu của từng branch
    # .alias("Branch_Revenue"): đổi tên cột kết quả
    F_sum("Revenue").alias("Branch_Revenue"),
    # F_avg("Rating"): tính rating trung bình
    F_avg("Rating").alias("Avg_Rating"),
    # F_count("*"): đếm tổng số hóa đơn (* = tất cả dòng)
    F_count("*").alias("Num_Invoices"),
    # F_avg("Profit"): tính lợi nhuận trung bình mỗi giao dịch
    F_avg("Profit").alias("Avg_Profit")
    # orderBy(...desc()): sắp xếp giảm dần theo doanh thu (branch có doanh thu cao nhất ở đầu)
).orderBy(col("Branch_Revenue").desc())

# Hiển thị kết quả thống kê
branch_stats.show()
```

> **💡 Tip**: `branch_stats.toPandas()` để chuyển sang Pandas và vẽ biểu đồ dễ dàng.

---

## 5. Phân cụm Khách hàng (K-Means)

### BƯỚC 1: Tạo Features Vector

```python
# VectorAssembler: ghép 4 cột số thành vector [UnitPrice, Quantity, Rating, Revenue]
assembler = VectorAssembler(
    inputCols=["UnitPrice", "Quantity", "Rating", "Revenue"],  # 4 features
    outputCol="features",  # Tên cột vector: mỗi dòng có vector [price, qty, rating, revenue]
    handleInvalid="skip"
)
df_vector = assembler.transform(df_clean).select("features")
```

### BƯỚC 2: Chuẩn hóa Features

```python
# StandardScaler: chuẩn hóa về mean=0, std=1
# Lý do: K-Means dùng khoảng cách Euclidean → features có đơn vị khác nhau cần scale
scaler = StandardScaler(
    inputCol="features",
    outputCol="scaledFeatures",
    withStd=True,  # Scale bằng std
    withMean=False  # Không center về mean (an toàn với bộ nhớ)
)
scaler_model = scaler.fit(df_vector)
df_scaled = scaler_model.transform(df_vector)
```

### BƯỚC 3: Tìm k tối ưu (Silhouette Score)

```python
# Silhouette Score: đo mức độ phân tách giữa các cụm (càng cao càng tốt, max=1)
evaluator = ClusteringEvaluator(featuresCol="scaledFeatures", predictionCol="cluster")

silhouette_scores = []
for k in range(2, 11):
    kmeans = KMeans(k=k, seed=42, featuresCol="scaledFeatures", predictionCol="cluster")
    model = kmeans.fit(df_scaled)
    predictions_temp = model.transform(df_scaled)
    score = evaluator.evaluate(predictions_temp)
    silhouette_scores.append(score)
    print(f"For k={k}, Silhouette Score: {score:.3f}")

best_k = silhouette_scores.index(max(silhouette_scores)) + 2
print(f"Best k (số cụm tối ưu): {best_k}")
```

### BƯỚC 4: Train K-Means & Predict

```python
# Train model chính thức với k tối ưu
kmeans_final = KMeans(k=best_k, seed=42, featuresCol="scaledFeatures", predictionCol="cluster")
kmeans_model = kmeans_final.fit(df_scaled)

# Predict và đánh giá
df_clustered = kmeans_model.transform(df_scaled)
final_silhouette = evaluator.evaluate(df_clustered)
print(f"Final Silhouette Score với k={best_k}: {final_silhouette:.3f}")

# Phân bố khách hàng theo cụm
df_clustered.groupBy("cluster").count().orderBy("cluster").show()
```

---

## 6. Dự báo Doanh thu (Linear Regression)

### BƯỚC 1: Tạo dữ liệu Time Series

```python
# Group theo ngày và tính tổng doanh thu
df_daily_agg = df_clean.groupBy("RealDate").agg(F_sum("Revenue").alias("DailyRevenue"))

# Tạo chỉ số ngày (DayIndex)
window_spec = Window.orderBy("RealDate")
df_daily = df_daily_agg.withColumn("DayIndex", row_number().over(window_spec))
```

### BƯỚC 2-5: Train Linear Regression

```python
# Tạo features vector
assembler_ts = VectorAssembler(inputCols=["DayIndex"], outputCol="features")
df_ts_vec = assembler_ts.transform(df_daily.select("DayIndex", "DailyRevenue"))

# Chia train/test
train_data, test_data = df_ts_vec.randomSplit([0.8, 0.2], seed=42)

# Train model
lr = LinearRegression(
    featuresCol="features",
    labelCol="DailyRevenue",
    maxIter=100,
    regParam=0.01
)
lr_model = lr.fit(train_data)

# Predict & RMSE
predictions = lr_model.transform(test_data)
from pyspark.ml.evaluation import RegressionEvaluator
rmse_evaluator = RegressionEvaluator(labelCol="DailyRevenue", predictionCol="prediction", metricName="rmse")
rmse = rmse_evaluator.evaluate(predictions)
print(f"RMSE: {rmse:.2f}")
```

### BƯỚC 8: Dự báo 10 ngày tương lai

```python
# Tìm ngày cuối cùng
max_day = df_daily.agg({"DayIndex": "max"}).collect()[0][0]

# Tạo DataFrame 10 ngày tiếp theo
future_days = spark.createDataFrame([{"DayIndex": i} for i in range(max_day+1, max_day+11)])

# Dự báo
future_pred = assembler_ts.transform(future_days)
future_forecast = lr_model.transform(future_pred)
future_forecast.select("DayIndex", "prediction").show()
```

---

## 7. Export Kết quả

```python
# Export phân cụm
df_clustered.coalesce(1).write.mode("overwrite").option("header", "true").csv("customer_clusters")

# Export dự báo
predictions.coalesce(1).write.mode("overwrite").option("header", "true").csv("revenue_forecasts")

# Chuyển sang Pandas để visualize
branch_pd = branch_stats.toPandas()
clusters_pd = df_clustered.sample(0.1).toPandas()

print("✅ Exported!")

# Dừng Spark Session
spark.stop()
```

---
