# 4-Week Polars Study Plan (1 Hour / Day)

> **Environment Setup**: Your virtual environment is located in this directory (`/Users/dikshie/VIRTUAL/polars_study`).  
> Run Python scripts directly using `./bin/python <script.py>` or activate with `source bin/activate`.  
> Polars (`v1.44.2`) is already installed.

---

## 📅 Overview

| Week | Theme | Focus |
| :--- | :--- | :--- |
| **Week 1** | **Foundations & Expressions** | DataFrames, Series, Contexts (`select`, `with_columns`, `filter`), Expression building blocks |
| **Week 2** | **Transformations & Aggregations** | `group_by`, Aggregations, Window functions (`over`), Joins, Reshaping (`pivot`/`unpivot`) |
| **Week 3** | **Data Types, Temporal & I/O** | Missing values/Nulls, String & Categorical operations, Datetime/Time-series, Parquet/CSV I/O |
| **Week 4** | **Lazy Evaluation & Real-World Pipeline** | LazyFrames (`scan_*`), Query optimization (`explain`), Streaming, Capstone Mini-Project |

---

## 🗓️ Week 1: Foundations & The Expression System

### Day 1: Polars Architecture & Basics
- **Goal**: Understand Apache Arrow columnar format, Polars vs Pandas differences (no index, parallel by default, immutable schemas).
- **Theory (15m)**: Read official documentation [Quickstart](https://docs.pola.rs/user-guide/getting-started/).
- **Coding (25m)**:
  ```python
  import polars as pl

  data = {
      "name": ["Alice", "Bob", "Charlie", "David"],
      "age": [24, 30, 22, 35],
      "department": ["HR", "IT", "IT", "Finance"],
      "salary": [50000, 75000, 62000, 90000],
  }
  df = pl.DataFrame(data)
  print(df)
  print(df.schema)
  print(df.shape)
  ```
- **Exercise (20m)**: Create a Series and DataFrame from scratch, inspect column dtypes, and explore `.head()`, `.tail()`, and `.describe()`.

---

### Day 2: Selection & Projection (`select`)
- **Goal**: Master selecting columns and transforming them with expressions.
- **Theory (15m)**: Why Polars uses expressions instead of sequential column mutation.
- **Coding (25m)**:
  ```python
  # Select columns and calculate new expressions
  res = df.select(
      pl.col("name"),
      (pl.col("salary") * 1.1).alias("salary_with_bonus"),
      (pl.col("age") >= 30).alias("is_senior"),
  )
  print(res)

  # Select by pattern or type
  df.select(pl.col("^.*name$"), pl.col(pl.Int64))
  ```
- **Exercise (20m)**: Select all numerical columns, multiply them by 2, and rename with suffix `_doubled`.

---

### Day 3: Adding & Updating Columns (`with_columns`)
- **Goal**: Modify and add new columns without dropping existing ones.
- **Theory (15m)**: Understand parallel column evaluation in `with_columns`.
- **Coding (25m)**:
  ```python
  df_updated = df.with_columns(
      annual_bonus=pl.col("salary") * 0.15,
      name_upper=pl.col("name").str.to_uppercase(),
  )
  print(df_updated)
  ```
- **Exercise (20m)**: Add a column `tax_bracket` using `pl.when().then().otherwise()`.

---

### Day 4: Filtering Rows (`filter`)
- **Goal**: Filter rows efficiently using Boolean expressions.
- **Theory (15m)**: Combining conditions with `&`, `|`, `~` vs comma-separated conditions.
- **Coding (25m)**:
  ```python
  # Comma-separated acts as AND
  it_over_25 = df.filter(pl.col("department") == "IT", pl.col("age") > 25)

  # OR conditions
  hr_or_high_salary = df.filter(
      (pl.col("department") == "HR") | (pl.col("salary") > 80000)
  )
  ```
- **Exercise (20m)**: Filter rows where `name` starts with `'A'` OR `salary` is between 60,000 and 80,000.

---

### Day 5: Conditional Logic (`when - then - otherwise`)
- **Goal**: Write multi-branch conditions cleanly.
- **Theory (15m)**: Chaining `when().then().when().then().otherwise()`.
- **Coding (25m)**:
  ```python
  df_categorized = df.with_columns(
      tier=pl.when(pl.col("salary") > 80000)
      .then(pl.lit("Executive"))
      .when(pl.col("salary") >= 60000)
      .then(pl.lit("Senior"))
      .otherwise(pl.lit("Associate"))
  )
  print(df_categorized)
  ```
- **Exercise (20m)**: Categorize `age` into `"<25"`, `"25-34"`, `"35+"`.

---

### Day 6: Sorting & Deduplication
- **Goal**: Learn `.sort()` and `.unique()`.
- **Coding (35m)**:
  ```python
  df.sort("salary", descending=True)
  df.sort(by=["department", "salary"], descending=[False, True])
  df.unique(subset=["department"], keep="first")
  ```
- **Exercise (25m)**: Find the highest paid employee in each department using sort + unique or top-k idioms.

---

### Day 7: Weekly Review & Mini Challenge
- **Goal**: Consolidate Week 1 learning.
- **Challenge (60m)**: Create a synthetic customer transactions dataset (50 rows) and write a single expression pipeline that cleans names, computes discounts, filters inactive customers, and sorts by revenue.

---

## 🗓️ Week 2: Transformations, Aggregations & Joins

### Day 8: Group By & Basic Aggregations
- **Goal**: Group data and calculate summary statistics.
- **Theory (15m)**: `group_by()` context and returning aggregated scalar values per group.
- **Coding (25m)**:
  ```python
  summary = df.group_by("department").agg(
      pl.len().alias("headcount"),
      pl.col("salary").mean().alias("avg_salary"),
      pl.col("salary").max().alias("max_salary"),
  )
  print(summary)
  ```
- **Exercise (20m)**: Compute the median age and sum of salaries per department.

---

### Day 9: Advanced Aggregations (Lists & Conditions)
- **Goal**: Aggregate into lists and apply conditional aggregations within groups.
- **Coding (35m)**:
  ```python
  df.group_by("department").agg(
      pl.col("name").implode().alias("employee_list"),
      (pl.col("salary") > 60000).sum().alias("high_earners_count"),
  )
  ```
- **Exercise (25m)**: Aggregate employee names as a sorted list for each department.

---

### Day 10: Window Functions (`.over()`)
- **Goal**: Compute group-level metrics while retaining all original rows.
- **Theory (15m)**: The power of `.over()` without needing merge/join steps.
- **Coding (25m)**:
  ```python
  df_window = df.with_columns(
      dept_avg=pl.col("salary").mean().over("department"),
      dept_rank=pl.col("salary").rank(descending=True).over("department"),
  )
  print(df_window)
  ```
- **Exercise (20m)**: Calculate each employee's salary difference from their department average (`salary - dept_avg`).

---

### Day 11: Combining Data - Joins
- **Goal**: Master `inner`, `left`, `full`, `semi`, and `anti` joins.
- **Coding (35m)**:
  ```python
  dept_info = pl.DataFrame({
      "department": ["IT", "HR", "Marketing"],
      "manager": ["Eve", "Grace", "Heidi"],
  })

  # Left join
  df.join(dept_info, on="department", how="left")

  # Anti join (find departments in df that are not in dept_info)
  df.join(dept_info, on="department", how="anti")
  ```
- **Exercise (25m)**: Implement a lookup table join with mismatched column names using `left_on` and `right_on`.

---

### Day 12: Concatenation (`concat`)
- **Goal**: Combine DataFrames vertically, horizontally, and diagonally.
- **Coding (35m)**:
  ```python
  df1 = pl.DataFrame({"a": [1, 2], "b": [3, 4]})
  df2 = pl.DataFrame({"a": [5, 6], "b": [7, 8]})
  df_vertical = pl.concat([df1, df2], how="vertical")
  ```
- **Exercise (25m)**: Concatenate two DataFrames with overlapping but different columns using `how="diagonal"`.

---

### Day 13: Reshaping - Pivot & Unpivot
- **Goal**: Convert wide to long and long to wide data.
- **Coding (35m)**:
  ```python
  sales = pl.DataFrame({
      "date": ["2026-01-01", "2026-01-01", "2026-01-02"],
      "product": ["A", "B", "A"],
      "revenue": [100, 200, 150],
  })
  # Pivot (Long to Wide)
  pivoted = sales.pivot(on="product", values="revenue", index="date")

  # Unpivot (Wide to Long)
  unpivoted = pivoted.unpivot(index="date", variable_name="product", value_name="revenue")
  ```
- **Exercise (25m)**: Reshape a multi-metric wide table into a tidy long format.

---

### Day 14: Week 2 Mini-Project
- **Goal**: Build an Sales & Inventory Analytics pipeline.
- **Challenge (60m)**: Given mock tables `orders`, `products`, `customers`:
  1. Join all three tables.
  2. Compute customer lifetime value (LTV).
  3. Rank products within each category using `.over()`.
  4. Pivot monthly sales by category.

---

## 🗓️ Week 3: Data Types, Strings, Dates & I/O

### Day 15: Handling Missing Data & Nulls
- **Goal**: Understand Polars' strict Null handling (vs Pandas NaN/None confusion).
- **Coding (35m)**:
  ```python
  df_nulls = pl.DataFrame({
      "val": [10, None, 30, None, 50]
  })
  df_nulls.with_columns(
      filled_zero=pl.col("val").fill_null(0),
      filled_forward=pl.col("val").fill_null(strategy="forward"),
      is_missing=pl.col("val").is_null(),
  )
  ```
- **Exercise (25m)**: Drop nulls conditionally or impute missing values with group means using `.over()`.

---

### Day 16: String & Categorical Data (`.str`, `.cat`)
- **Goal**: Master string manipulation expressions and Enum/Categorical data types for memory efficiency.
- **Coding (35m)**:
  ```python
  names_df = pl.DataFrame({"full_name": ["John Doe", "Jane Smith", "Bob Vance"]})
  names_df.with_columns(
      first_name=pl.col("full_name").str.split(" ").list.get(0),
      last_name=pl.col("full_name").str.split(" ").list.get(1),
      contains_smith=pl.col("full_name").str.contains("Smith"),
      cat_name=pl.col("full_name").cast(pl.Categorical),
  )
  ```
- **Exercise (25m)**: Clean dirty email addresses: strip whitespace, lowercase, and extract domain name.

---

### Day 17: Temporal & Date/Time Manipulation (`.dt`)
- **Goal**: Parse, format, and manipulate timestamps and date ranges.
- **Coding (35m)**:
  ```python
  dates = pl.DataFrame({"timestamp": ["2026-01-15 08:30:00", "2026-02-20 14:45:00"]})
  parsed = dates.with_columns(
      dt=pl.col("timestamp").str.to_datetime("%Y-%m-%d %H:%M:%S")
  ).with_columns(
      year=pl.col("dt").dt.year(),
      month=pl.col("dt").dt.month(),
      day_name=pl.col("dt").dt.strftime("%A"),
      offset=pl.col("dt").dt.offset_by("1mo"),
  )
  ```
- **Exercise (25m)**: Group transactions by week or month and calculate rolling monthly metrics.

---

### Day 18: File I/O - CSV & JSON
- **Goal**: Learn fast I/O with `read_csv`, `write_csv`, `read_json`, `write_ndjson`.
- **Coding (35m)**:
  ```python
  df.write_csv("sample.csv")
  loaded_df = pl.read_csv("sample.csv", schema_overrides={"salary": pl.Float64})
  ```
- **Exercise (25m)**: Export data to NDJSON (Newline Delimited JSON) and read it back with custom types.

---

### Day 19: High-Performance I/O - Parquet
- **Goal**: Understand why Parquet is the standard storage format for Arrow/Polars.
- **Coding (35m)**:
  ```python
  df.write_parquet("data.parquet", compression="zstd")
  parquet_df = pl.read_parquet("data.parquet", columns=["name", "salary"])
  ```
- **Exercise (25m)**: Benchmark read/write time and file size between CSV vs Parquet.

---

### Day 20: Database & Arrow Interoperability
- **Goal**: Convert to/from PyArrow, NumPy, and Pandas.
- **Coding (35m)**:
  ```python
  arrow_table = df.to_arrow()
  from_arrow = pl.from_arrow(arrow_table)
  numpy_arr = df.select("salary").to_numpy()
  ```
- **Exercise (25m)**: Practice seamless data interchange between Polars and NumPy/Pandas.

---

### Day 21: Week 3 Review & ETL Drill
- **Goal**: Build an end-to-end data ingestion & normalization pipeline.
- **Challenge (60m)**: Ingest dirty CSV data, cast types, parse datetimes, impute nulls, and save to partitioned Parquet files.

---

## 🗓️ Week 4: Lazy Evaluation, Optimization & Capstone

### Day 22: Introduction to LazyFrames & Query Plans
- **Goal**: Understand `pl.scan_csv()` / `pl.scan_parquet()` and `.collect()`.
- **Theory (15m)**: Predicate pushdown (filters early) & Projection pushdown (reads only required columns).
- **Coding (25m)**:
  ```python
  lf = (
      pl.scan_parquet("data.parquet")
      .filter(pl.col("salary") > 60000)
      .select("name", "department", "salary")
  )
  print(lf.explain())  # View optimized query plan
  result = lf.collect()  # Execute
  ```
- **Exercise (20m)**: Compare execution plans of `.explain(optimized=True)` vs `.explain(optimized=False)`.

---

### Day 23: Streaming Engine for Larger-than-RAM Data
- **Goal**: Process datasets that exceed available RAM using Polars streaming.
- **Coding (35m)**:
  ```python
  # Process chunk by chunk out-of-core
  streamed_df = (
      pl.scan_parquet("data.parquet")
      .group_by("department")
      .agg(pl.col("salary").mean())
      .collect(engine="streaming")
  )
  ```
- **Exercise (25m)**: Test memory usage on a larger synthetic dataset using streaming vs eager mode.

---

### Day 24: Pandas Anti-Patterns & Polars Idioms
- **Goal**: Avoid row iteration (`apply`, `iterrows`) and write 100% vectorized Polars code.
- **Theory (15m)**: Why `map_elements()` should be a last resort.
- **Coding (25m)**:
  ```python
  # BAD: df.with_columns(pl.col("x").map_elements(lambda v: v * 2))
  # GOOD:
  df.with_columns(pl.col("x") * 2)
  ```
- **Exercise (20m)**: Refactor 3 typical Pandas snippets into idiomatic Polars expressions.

---

### Day 25: Polars SQL Context
- **Goal**: Query Polars DataFrames using standard SQL syntax.
- **Coding (35m)**:
  ```python
  ctx = pl.SQLContext()
  ctx.register("employees", df)
  sql_result = ctx.execute(
      """
      SELECT department, AVG(salary) as avg_sal
      FROM employees
      GROUP BY department
      HAVING AVG(salary) > 60000
      """
  ).collect()
  print(sql_result)
  ```
- **Exercise (25m)**: Write SQL queries joining multiple registered Polars tables.

---

### Day 26 & 27: Capstone Project
- **Goal**: Build a complete, production-ready Polars data pipeline.
- **Project Requirements**:
  1. Generate or load a realistic dataset (>100k rows: e.g. e-commerce transactions, web logs, or financial data).
  2. Implement a Lazy pipeline with schema validation.
  3. Complex feature engineering:
     - Rolling windows (`rolling_*`)
     - Group aggregations with `.over()`
     - String parsing and datetime feature extraction
  4. Optimize query and output partitioned Parquet files.
  5. Inspect execution plan with `explain()`.

---

### Day 28: Graduation & Next Steps
- **Goal**: Review, test, and explore advanced topics (Polars Plugins in Rust, GPU support via `polars-gpu`).
- **Activity (60m)**: Code review your capstone project, verify execution speeds, and explore [Polars Plugin Documentation](https://marcogorelli.github.io/polars-plugins-tutorial/).

---

## 💡 Daily 1-Hour Study Routine
1. **00 - 15m**: Read documentation / reference guide for the day's topic.
2. **15 - 40m**: Run and experiment with the day's code snippet in `./bin/python`.
3. **40 - 60m**: Solve the day's exercise and verify the output.
