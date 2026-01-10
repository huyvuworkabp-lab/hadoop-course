---
title: "BÀI THỰC HÀNH PYSPARK: PHÂN CỤM KHÁCH HÀNG (CUSTOMER SEGMENTATION)"
description: "Hướng dẫn hoàn chỉnh: Data Analysis → Feature Engineering → K-Means Clustering → Visualization (Google Colab ready)"
author: "Điền tên bạn"
date: "2026-01-10"
version: "1.0"
tags: [pyspark, spark, kmeans, customer-segmentation, data-analysis, ml]
---

<div align="center">

# PYSPARK CUSTOMER SEGMENTATION  
## K-MEANS CLUSTERING

[![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![PySpark](https://img.shields.io/badge/PySpark-3.5+-F8961E?style=for-the-badge&logo=apachespark&logoColor=white)](https://spark.apache.org)
[![Google Colab](https://img.shields.io/badge/Google%20Colab-FEE71C?style=for-the-badge&logo=google-colab&logoColor=white)](https://colab.research.google.com)

**Dataset**: Customer data (ID, Age, Income, Sex, MaritalStatus, Education, Occupation, SettlementSize...)

**Mục tiêu**: Phân cụm khách hàng thành 3-5 nhóm → **phân tích profile từng nhóm** → **gợi ý marketing strategy**.

</div>

---

## 📋 Mục lục

- [1. Setup Spark](#1-setup-spark)
- [2. Load & Chuẩn bị dữ liệu](#2-load--chuẩn-bị-dữ-liệu)
- [3. Data Analysis Pipeline](#3-data-analysis-pipeline)
- [4. Clustering Techniques](#4-clustering-techniques-k-means)
- [5. Phân tích chi tiết từng nhóm](#5-phân-tích-chi-tết-từng-nhóm)
- [6. Visualization](#6-visualization)

---

## 1. Setup Spark

```python
# Cài đặt PySpark (Colab 2026 chỉ cần thế này)
!pip install pyspark findspark -q  # Cài PySpark + findspark để Colab nhận Spark

import findspark
findspark.init()  # Khởi động môi trường Spark cho Python

from pyspark.sql import SparkSession  # Tạo SparkSession để làm việc với DataFrame
from pyspark.sql.functions import col, avg, count, min, max  # Một số hàm phân tích cơ bản

# Tạo SparkSession (ứng dụng Spark chính)
spark = SparkSession.builder.appName("CustomerSegmentation").getOrCreate()
spark  # Hiển thị thông tin SparkSession
```

---

## 2. Load & Chuẩn bị dữ liệu

```python
# Cách 1: Upload file từ máy
from google.colab import files  # Thư viện giúp upload file từ máy lên Colab
uploaded = files.upload()  # Chạy cell này → chọn file customers.csv

# Đọc CSV vào DataFrame Spark
df = spark.read.option("header", "true").option("inferSchema", "true").csv("customers.csv")  # header, tự đoán kiểu dữ liệu
df.show(5)  # Xem 5 dòng đầu
df.printSchema()  # Xem schema (kiểu dữ liệu từng cột)
```

**Chuẩn hoá tên cột & xử lý dữ liệu cơ bản:**

```python
from pyspark.sql.functions import trim

# Đổi tên cột bỏ khoảng trắng để code dễ hơn
df = df.withColumnRenamed("Marital status", "MaritalStatus") \
       .withColumnRenamed("Settlement size", "SettlementSize")

# Loại bỏ khoảng trắng dư trong string (Sex, MaritalStatus, Education, Occupation, SettlementSize)
str_cols = ["Sex", "MaritalStatus", "Education", "Occupation", "SettlementSize"]
for c in str_cols:
    df = df.withColumn(c, trim(col(c)))  # trim: xóa space đầu/cuối

# Bỏ những dòng thiếu dữ liệu quan trọng
df = df.dropna(subset=["Age", "Income", "Education", "Sex", "MaritalStatus", "SettlementSize"])
df.show(5)
```

---

## 3. Data Analysis Pipeline

### 3.1. Thống kê mô tả cơ bản

```python
# Thống kê tổng quan cho Age và Income
df.describe(["Age", "Income"]).show()  # count, mean, stddev, min, max cho Age & Income

# Số lượng khách hàng theo giới tính
df.groupBy("Sex").agg(count("*").alias("CustomerCount")).show()

# Số lượng theo tình trạng hôn nhân
df.groupBy("MaritalStatus").agg(count("*").alias("CustomerCount")).show()
```

### 3.2. Phân tích theo nhóm

```python
# Thu nhập trung bình theo trình độ học vấn
df.groupBy("Education").agg(avg("Income").alias("AvgIncome")).orderBy(col("AvgIncome").desc()).show()

# Tuổi trung bình theo kích thước khu định cư (SettlementSize)
df.groupBy("SettlementSize").agg(avg("Age").alias("AvgAge")).orderBy("SettlementSize").show()
```

### 3.3. Feature Engineering cho ML

```python
from pyspark.ml.feature import StringIndexer, VectorAssembler, StandardScaler

# Biến categorical → numerical (index)
indexers = [
    StringIndexer(inputCol="Sex", outputCol="SexIndex"),
    StringIndexer(inputCol="MaritalStatus", outputCol="MaritalStatusIndex"),
    StringIndexer(inputCol="Education", outputCol="EducationIndex"),
    StringIndexer(inputCol="Occupation", outputCol="OccupationIndex"),
    StringIndexer(inputCol="SettlementSize", outputCol="SettlementSizeIndex")
]
for idx in indexers:
    df = idx.fit(df).transform(df)  # fit + transform từng cột, tạo thêm cột Index

# Chọn các feature để clustering
feature_cols = ["Age", "Income", "SexIndex", "MaritalStatusIndex", "EducationIndex", "OccupationIndex", "SettlementSizeIndex"]

# VectorAssembler: gộp nhiều cột feature thành 1 vector "features"
assembler = VectorAssembler(inputCols=feature_cols, outputCol="features")
df_features = assembler.transform(df)  # Thêm cột features vào DataFrame
df_features.select("ID", "features").show(5, truncate=False)
```

**Chuẩn hóa dữ liệu (rất quan trọng cho K-means):**

```python
# StandardScaler: đưa tất cả feature về cùng thang đo (mean≈0, std≈1)
scaler = StandardScaler(inputCol="features", outputCol="scaledFeatures", withStd=True, withMean=True)
scaler_model = scaler.fit(df_features)  # học tham số scale từ data
df_scaled = scaler_model.transform(df_features)  # tạo cột scaledFeatures đã chuẩn hóa
df_scaled.select("ID", "scaledFeatures").show(5, truncate=False)
```

---

## 4. Clustering Techniques (K-Means)

### 4.1. Train mô hình K-Means

```python
from pyspark.ml.clustering import KMeans
from pyspark.ml.evaluation import ClusteringEvaluator

# Khởi tạo model KMeans với k=4 nhóm (tuỳ bạn chọn 3--5 để demo)
k = 4  # số cluster
kmeans = KMeans(featuresCol="scaledFeatures", predictionCol="cluster", k=k, seed=42)  # seed để kết quả ổn định

# Train model trên dữ liệu đã scale
kmeans_model = kmeans.fit(df_scaled)

# Gán cluster cho từng khách hàng
df_clustered = kmeans_model.transform(df_scaled)  # thêm cột 'cluster'
df_clustered.select("ID", "Age", "Income", "cluster").show(10)
```

**Đánh giá chất lượng clustering:**

```python
# ClusteringEvaluator dùng Silhouette score để đo chất lượng cluster (0-1, càng cao càng tốt)
evaluator = ClusteringEvaluator(featuresCol="scaledFeatures", predictionCol="cluster", metricName="silhouette")
silhouette = evaluator.evaluate(df_clustered)
print(f"Silhouette score (chất lượng phân cụm): {silhouette:.3f}")

# Đếm số khách hàng trong từng cụm
df_clustered.groupBy("cluster").agg(count("*").alias("CustomerCount")).orderBy("cluster").show()
```

---

## 5. Phân tích chi tiết từng nhóm

### 5.1. Thống kê mô tả theo cluster

```python
# Thống kê Age & Income trung bình theo cluster
cluster_stats = df_clustered.groupBy("cluster").agg(
    avg("Age").alias("AvgAge"),
    avg("Income").alias("AvgIncome"),
    count("*").alias("CustomerCount")
).orderBy("cluster")
cluster_stats.show()
```

**Phân bố categorical trong mỗi nhóm:**

```python
# Tỷ lệ Sex trong mỗi cluster
df_clustered.groupBy("cluster", "Sex").agg(count("*").alias("cnt")) \
    .orderBy("cluster", "cnt", ascending=[True, False]) \
    .show()

# Tỷ lệ MaritalStatus trong mỗi cluster
df_clustered.groupBy("cluster", "MaritalStatus").agg(count("*").alias("cnt")) \
    .orderBy("cluster", "cnt", ascending=[True, False]) \
    .show()
```

### 5.2. Gợi ý phân tích cho từng nhóm

**Ví dụ mô tả profile (dựa vào kết quả `cluster_stats`):**

- **Cluster 0**: Trẻ (AvgAge ~25), thu nhập thấp → "Young Budget Shoppers"
- **Cluster 1**: Trung niên (AvgAge ~40), thu nhập cao → "Family Premium Buyers"  
- **Cluster 2**: Cao tuổi (AvgAge ~55), thu nhập trung bình → "Senior Loyalists"

**Marketing Strategy**:
```
Cluster 0 → Discount promotions, entry-level products
Cluster 1 → Premium brands, family packages  
Cluster 2 → Loyalty programs, health products
```

---

## 6. Visualization (Pandas + Matplotlib)

```python
# Lấy bảng thống kê cluster_stats sang Pandas để vẽ biểu đồ
cluster_stats_pd = cluster_stats.toPandas()  # chuyển DataFrame Spark → DataFrame Pandas
cluster_stats_pd  # xem dữ liệu trong môi trường Pandas
```

**Biểu đồ cột:**

```python
import matplotlib.pyplot as plt

# Biểu đồ cột: Thu nhập trung bình theo cluster
plt.figure(figsize=(8, 5))
plt.bar(cluster_stats_pd["cluster"], cluster_stats_pd["AvgIncome"])
plt.xlabel("Cluster")  # trục X: mã cluster
plt.ylabel("Avg Income")  # trục Y: thu nhập trung bình
plt.title("Average Income per Cluster")  # tiêu đề
plt.show()

# Biểu đồ cột: Tuổi trung bình theo cluster
plt.figure(figsize=(8, 5))
plt.bar(cluster_stats_pd["cluster"], cluster_stats_pd["AvgAge"], color="orange")
plt.xlabel("Cluster")
plt.ylabel("Avg Age")
plt.title("Average Age per Cluster")
plt.show()
```

**Scatter Plot 2D (Age vs Income, màu theo cluster):**

```python
# Lấy sample nhỏ để vẽ scatter plot
sample_pd = df_clustered.select("Age", "Income", "cluster").sample(0.2, seed=42).toPandas()  # lấy 20% random

plt.figure(figsize=(8, 6))
scatter = plt.scatter(sample_pd["Age"], sample_pd["Income"], c=sample_pd["cluster"], cmap="viridis")
plt.xlabel("Age")
plt.ylabel("Income")
plt.title("Age vs Income by Cluster")
plt.colorbar(scatter, label="Cluster")  # chú giải màu theo cluster
plt.show()
```

---

```python
spark.stop()  # Giải phóng tài nguyên Spark
```

<div align="center">

**🎉 Hoàn thành Customer Segmentation!**

> **Next Steps**:
> - Export clusters: `df_clustered.coalesce(1).write.csv("clusters")`
> - Thử k=3,5,6 → so sánh Silhouette Score
> - Thêm features: Spending Score, Product Category

</div>
