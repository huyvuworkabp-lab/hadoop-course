---
title: "GIẢI BÀI TẬP PYSPARK – THAO TÁC DATAFRAME VÀ SQL"
author: "(Điền tên bạn)"
version: "1.0"
updated: "2026-01-06"
tags: [pyspark, spark, dataframe, sql, exercises]
---

<div align="center">

# GIẢI BÀI TẬP PYSPARK – THAO TÁC DATAFRAME VÀ SQL

*Tài liệu tham khảo lời giải chi tiết cho các bài tập thực hành thao tác dữ liệu Supermarket Sales.*

</div>

**Tác giả:** Điền tên bạn  
**Phiên bản:** 1.0  
**Ngày cập nhật:** 2026-01-06

---

## Mục lục

- [Lưu ý](#lưu-ý)
- [Phần 1: Thao tác cơ bản với DataFrame](#phần-1-thao-tác-cơ-bản-với-dataframe)
- [Phần 2: Thống kê mô tả (Statistics)](#phần-2-thống-kê-mô-tả-statistics)
- [Phần 3: GroupBy & Aggregation](#phần-3-groupby--aggregation)
- [Phần 4: Spark SQL](#phần-4-spark-sql)

---

## Lưu ý

> **Note**: Giả định bạn đã chạy bước Setup Spark và có DataFrame `df_sales` từ các bước trước (đã load file `Supermarket.csv`).

---

## Phần 1: Thao tác cơ bản với DataFrame

```python
# 1. In schema, 10 dòng đầu, đếm số dòng/cột
df_sales.printSchema()
df_sales.show(10)
print(f"Rows: {df_sales.count()}, Columns: {len(df_sales.columns)}")

# 2. Chọn 5 cột, hiển thị 20 dòng
df_part2 = df_sales.select("Invoice ID", "Branch", "City", "Product line", "cogs")
df_part2.show(20)

# 3. Lọc Branch="A" và Payment="Cash"
df_sales.filter((col("Branch") == "A") & (col("Payment") == "Cash")).show()

# 4. Lọc Quantity >= 8 và Unit price > 50
df_sales.filter((col("Quantity") >= 8) & (col("Unit price") > 50)).show()

# 5. Sắp xếp giảm dần theo cogs, lấy top 15
df_sales.orderBy(col("cogs").desc()).show(15)

# 6. Tạo cột Total
df_sales = df_sales.withColumn("Total", col("Unit price") * col("Quantity"))
df_sales.select("Invoice ID", "Total").show(5)

# 7. Tạo cột Gross_Profit
df_sales = df_sales.withColumn("Gross_Profit", col("Total") - col("cogs"))
df_sales.select("Invoice ID", "Total", "cogs", "Gross_Profit").show(10)

# 8. Tạo cột Hour từ Time và thống kê
from pyspark.sql.functions import hour
# Lưu ý: Cột Time trong csv thường được Spark đọc là Timestamp. Nếu chưa, cần cast.
df_sales = df_sales.withColumn("Hour", hour(col("Time")))
df_sales.groupBy("Hour").count().orderBy("Hour").show()

# 9. Chuẩn hoá: bỏ null ở Unit price hoặc Quantity
df_clean = df_sales.dropna(subset=["Unit price", "Quantity"])
print("Cleaned count:", df_clean.count())

# 10. Lấy danh sách distinct
df_sales.select("Branch").distinct().show()
df_sales.select("City").distinct().show()
df_sales.select("Payment").distinct().show()
df_sales.select("Product line").distinct().show()
```

---

## Phần 2: Thống kê mô tả (Statistics)

```python
# 1. Describe các cột số
df_sales.describe(["Unit price", "Quantity", "cogs"]).show()

# 2. Describe cột Total
df_sales.describe(["Total"]).show()

# 3. Min/Max/Avg của Unit price
from pyspark.sql.functions import min, max, avg, stddev, corr
df_sales.select(min("Unit price"), max("Unit price"), avg("Unit price")).show()

# 4. Min/Max/Avg của cogs theo Branch
df_sales.groupBy("Branch").agg(min("cogs"), max("cogs"), avg("cogs")).show()

# 5. Đếm theo Gender
df_sales.groupBy("Gender").count().show()

# 6. Median xấp xỉ (approxQuantile)
median_price = df_sales.approxQuantile("Unit price", [0.5], 0.01)[0]
median_cogs = df_sales.approxQuantile("cogs", [0.5], 0.01)[0]
print(f"Median Price: {median_price}, Median COGS: {median_cogs}")

# 7. Stddev của Quantity theo Product line
df_sales.groupBy("Product line").agg(stddev("Quantity").alias("std_qty")).show()

# 8. Tương quan giữa Unit price và Quantity
correlation = df_sales.stat.corr("Unit price", "Quantity")
print(f"Correlation: {correlation}")
```

---

## Phần 3: GroupBy & Aggregation

```python
# 1. GroupBy Branch: Sum Qty, Sum cogs, Count
df_sales.groupBy("Branch").agg(
    F_sum("Quantity").alias("Total_Qty"),
    F_sum("cogs").alias("Total_COGS"),
    F_count("*").alias("Count")
).show()

# 2. Top 3 City theo Total
df_sales.groupBy("City").agg(F_sum("Total").alias("Revenue")) \
    .orderBy(col("Revenue").desc()).limit(3).show()

# 3. Product line bán chạy nhất (theo Qty)
df_sales.groupBy("Product line").agg(F_sum("Quantity").alias("Total_Qty")) \
    .orderBy(col("Total_Qty").desc()).limit(1).show()

# 4. GroupBy Payment: Sum & Avg Total
df_sales.groupBy("Payment").agg(
    F_sum("Total").alias("Total_Revenue"),
    F_avg("Total").alias("Avg_Revenue")
).orderBy(col("Total_Revenue").desc()).show()

# 5. Customer type stats
df_sales.groupBy("Customer type").agg(
    F_sum("Total").alias("Total_Rev"),
    F_avg("Unit price").alias("Avg_Price"),
    F_count("*").alias("Count")
).show()

# 6. Branch + Gender
df_sales.groupBy("Branch", "Gender").agg(
    F_sum("Total").alias("Total_Rev"),
    F_count("*").alias("Count")
).orderBy(col("Total_Rev").desc()).show()

# 7. City + Payment phổ biến nhất
df_sales.groupBy("City", "Payment").count().orderBy(col("count").desc()).show()

# 8. Ngày doanh thu cao nhất
df_sales.groupBy("Date").agg(F_sum("Total").alias("Daily_Rev")) \
    .orderBy(col("Daily_Rev").desc()).limit(1).show()

# 9. Member -> Product line revenue
df_sales.filter(col("Customer type") == "Member") \
    .groupBy("Product line").agg(F_sum("Total").alias("Member_Revenue")).show()

# 10. Top 5 hóa đơn theo Gross_Profit
df_sales.select("Invoice ID", "Branch", "Gross_Profit") \
    .orderBy(col("Gross_Profit").desc()).limit(5).show()
```

---

## Phần 4: Spark SQL

```python
# Đăng ký view
df_sales.createOrReplaceTempView("sales")

# 1. Đếm số bản ghi theo Branch
spark.sql("SELECT Branch, COUNT(*) FROM sales GROUP BY Branch").show()

# 2. Top 5 Product line theo cogs
spark.sql("""
    SELECT `Product line`, SUM(cogs) as Total_COGS
    FROM sales
    GROUP BY `Product line`
    ORDER BY Total_COGS DESC
    LIMIT 5
""").show()

# 3. Avg Price & Qty theo City
spark.sql("""
    SELECT City, AVG(`Unit price`) as Avg_Price, AVG(Quantity) as Avg_Qty
    FROM sales
    GROUP BY City
""").show()

# 4. Payment='Ewallet' & Member -> Sum Total theo Branch
spark.sql("""
    SELECT Branch, SUM(Total) as Branch_Revenue
    FROM sales
    WHERE Payment = 'Ewallet' AND `Customer type` = 'Member'
    GROUP BY Branch
""").show()

# 5. Hóa đơn có Unit price > Avg toàn bộ (Subquery)
spark.sql("""
    SELECT `Invoice ID`, `Unit price`
    FROM sales
    WHERE `Unit price` > (SELECT AVG(`Unit price`) FROM sales)
""").show(5)

# 6. Doanh thu theo giờ (dùng hàm HOUR trong SQL)
spark.sql("""
    SELECT HOUR(Time) as Sales_Hour, SUM(Total) as Hourly_Revenue
    FROM sales
    GROUP BY Sales_Hour
    ORDER BY Sales_Hour
""").show()

# 7. Product line có Sum Qty lớn nhất theo từng Branch
# (Cách dùng Window Function)
spark.sql("""
    WITH Ranked AS (
        SELECT Branch, `Product line`, SUM(Quantity) as Sum_Qty,
        RANK() OVER (PARTITION BY Branch ORDER BY SUM(Quantity) DESC) as rnk
        FROM sales
        GROUP BY Branch, `Product line`
    )
    SELECT * FROM Ranked WHERE rnk = 1
""").show()

# 8. Tỷ lệ % số hóa đơn theo Payment
spark.sql("""
    SELECT Payment, COUNT(*) as Pay_Count,
           COUNT(*) * 100.0 / (SELECT COUNT(*) FROM sales) as Percent
    FROM sales
    GROUP BY Payment
""").show()

# 9. CTE: Top 10 Gross Profit
spark.sql("""
    WITH Profit_Data AS (
        SELECT `Invoice ID`, (Quantity * `Unit price`) as Total,
               ((Quantity * `Unit price`) - cogs) as Gross_Profit
        FROM sales
    )
    SELECT * FROM Profit_Data ORDER BY Gross_Profit DESC LIMIT 10
""").show()

# 10. Check trùng Invoice ID
spark.sql("""
    SELECT `Invoice ID`, COUNT(*)
    FROM sales
    GROUP BY `Invoice ID`
    HAVING COUNT(*) > 1
""").show()
```
