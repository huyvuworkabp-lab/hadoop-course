# BÀI THỰC HÀNH PYSPARK CƠ BẢN

**Tác giả:** Trần Huy Vũ

**Phiên bản:** 1.0  
**Ngày cập nhật:** 06 Tháng 01, 2026 

---

## Mục lục
- [Yêu cầu chung](#yêu-cầu-chung)
- [1. Setup Spark](#1-setup-spark-colab-ready)
- [2. Tải file CSV](#2-tải-file-csv-3-cách---chọn-1)
  - [Cách 1: Upload file local](#cách-1-upload-file-local)
  - [Cách 2: Google Drive mount](#cách-2-google-drive-mount)
  - [Cách 3: Direct Drive link](#cách-3-direct-drive-link-file-id-bạn)
- [3. Xử lý DataFrame](#3-xử-lý-dataframe-cơ-bản)
- [4. Thống kê cơ bản](#4-thống-kê-cơ-bản-với-spark)
- [5. GroupBy & Aggregation](#5-groupby--aggregation)
- [6. Spark SQL](#6-spark-sql-cơ-bản)
- [7. Tổng hợp bài tập](#7-tổng-hợp-bài-tập)

---

## Yêu cầu chung
- Google Colab hoặc môi trường có Java + PySpark.
- File `Supermarket.csv` theo đúng đường dẫn ở phần “Tải file”.

Core dataset: File CSV (Invoice ID, Branch, City, Customer type, Gender, Product line, Unit price, Quantity, Date, Time, Payment, cogs)

---
## **1. Setup Spark (Colab-ready)**

```python
# 3 dòng duy nhất cần thiết (2026 Colab)
!pip install pyspark findspark -q
import findspark
findspark.init()

from pyspark.sql import SparkSession
from pyspark.sql.functions import col, sum as F_sum, avg as F_avg, count as F_count, max as F_max, round

spark = SparkSession.builder.appName("Supermarket-Sales").getOrCreate()
spark
```
---
## **2. Tải file CSV (3 cách - chọn 1)**

### **Cách 1: Upload file local**

```python
from google.colab import files
uploaded = files.upload()  # Upload Supermarket.csv
df_sales = spark.read.option("header", "true").csv("Supermarket.csv", inferSchema=True)
df_sales.show(5)
```

### **Cách 2: Google Drive mount**

```python
from google.colab import drive
drive.mount('/content/drive')
df_sales = spark.read.option("header", "true") \
    .csv("/content/drive/MyDrive/Supermarket.csv", inferSchema=True)  # Thay tên file
df_sales.show(5)
```

### **Cách 3: Direct Drive link (file ID bạn)**

```python
file_id = "1Fa7SxYpLDUjx2-lbjMhx1udDJd4ZZOMW"
!wget -q "https://drive.google.com/uc?export=download&id={file_id}" -O supermarket.csv
df_sales = spark.read.option("header", "true").csv("supermarket.csv", inferSchema=True)
df_sales.show(5)
```

Kiểm tra schema:

```python
df_sales.printSchema()
print("Total rows:", df_sales.count())
```
---
## **3. Xử lý DataFrame cơ bản**

```python
# Select + Filter + Sort
df_sales.select("Branch", "Gender", "cogs", "Quantity") \
    .filter((col("Branch") == "A") & (col("Quantity") > 5)) \
    .orderBy(col("cogs").desc()) \
    .show(10)

# Tạo cột mới: Total = Unit price * Quantity
df_sales.withColumn("Total", col("Unit price") * col("Quantity")) \
    .select("Invoice ID", "Product line", "Total") \
    .show(5)
```
---
## **4. Thống kê cơ bản với Spark**

```python
# DESCRIBE() - Summary stats (mean, stddev, min, max)
df_sales.describe(["Unit price", "Quantity", "cogs"]).show()

# Count theo category
print("Invoices by Branch:")
df_sales.groupBy("Branch").count().show()

print("Payment methods:")
df_sales.groupBy("Payment").count().show()
```

```python
# Stats có điều kiện
df_sales.filter(col("Gender") == "Female") \
    .describe(["Unit price", "cogs"]).show()
```
---
## **5. GroupBy & Aggregation**

```python
# GroupBy Branch: Tổng cogs, Avg price, Count
df_sales.groupBy("Branch") \
    .agg(
        F_sum("cogs").alias("Total_COGS"),
        F_avg("Unit price").alias("Avg_Price"),
        F_count("*").alias("Invoice_Count")
    ).orderBy(col("Total_COGS").desc()).show()
```

```python
# Multi-group: Product line + Gender
df_sales.groupBy("Product line", "Gender") \
    .agg(
        F_sum("Quantity").alias("Total_Qty"),
        round(F_avg("cogs"), 2).alias("Avg_COGS")
    ).show(10)
```
---
## **6. Spark SQL cơ bản**

```python
# Tạo temp view
df_sales.createOrReplaceTempView("sales")

# SQL cơ bản
spark.sql("""
    SELECT Branch, COUNT(*) as Count,
           ROUND(AVG(`Unit price`), 2) as Avg_Price
    FROM sales
    GROUP BY Branch
""").show()
```

```python
# SQL filter + aggregate
spark.sql("""
    SELECT `Product line`, SUM(cogs) as Total_COGS,
           COUNT(*) as Orders
    FROM sales
    WHERE `Customer type` = 'Member'
    GROUP BY `Product line`
    ORDER BY Total_COGS DESC
    LIMIT 5
""").show()
```
---
## **7. TỔNG HỢP BÀI TẬP**

### **Bài 1: Tạo cột Gross Profit = Total - cogs, top 5 Branch theo Gross Profit**

```python
df_sales.withColumn("Total", col("Unit price") * col("Quantity")) \
    .withColumn("Gross_Profit", col("Total") - col("cogs")) \
    .groupBy("Branch") \
    .agg(F_sum("Gross_Profit").alias("Total_Profit")) \
    .orderBy(col("Total_Profit").desc()) \
    .show()
```

### **Bài 2: SQL - Avg Quantity theo Payment method ở City = "Yangon"**

```python
spark.sql("""
    SELECT Payment, ROUND(AVG(Quantity), 2) as Avg_Qty
    FROM sales
    WHERE City = 'Yangon'
    GROUP BY Payment
    ORDER BY Avg_Qty DESC
""").show()
```

### **Bài 3: Stats - Describe Unit price theo Gender = "Male"**

```python
df_sales.filter(col("Gender") == "Male") \
    .describe("Unit price").show()
```

### **Bài 4: GroupBy "Customer type" + "Product line": Total Revenue, Avg Order Value**

```python
df_sales.withColumn("Total", col("Unit price") * col("Quantity")) \
    .groupBy("Customer type", "Product line") \
    .agg(
        F_sum("Total").alias("Total_Revenue"),
        round(F_avg("Total"), 2).alias("Avg_Order_Value")
    ).show(10)
```

### **Bài 5: Product line bán nhiều Quantity nhất cho Female customers ở Branch B**

```python
df_sales.filter((col("Gender") == "Female") & (col("Branch") == "B")) \
    .groupBy("Product line") \
    .agg(F_sum("Quantity").alias("Total_Qty")) \
    .orderBy(col("Total_Qty").desc()) \
    .limit(1) \
    .show()
```

### **Mini-project: Top 3 City theo Total Revenue của Members**

```python
# Tạo view với Total column
df_analysis = df_sales.withColumn("Total", col("Unit price") * col("Quantity"))
df_analysis.createOrReplaceTempView("sales_analysis")

spark.sql("""
    SELECT City, ROUND(SUM(Total), 2) as City_Revenue
    FROM sales_analysis
    WHERE `Customer type` = 'Member'
    GROUP BY City
    ORDER BY City_Revenue DESC
    LIMIT 3
""").show()

# Export kết quả
spark.sql("""
    SELECT City, ROUND(SUM(Total), 2) as City_Revenue
    FROM sales_analysis
    WHERE `Customer type` = 'Member'
    GROUP BY City
    ORDER BY City_Revenue DESC
    LIMIT 3
""").coalesce(1).write.mode("overwrite").option("header", "true").csv("top_cities")
```

```python
# Kết thúc session
spark.stop()
```
