
# **Task 3: Buyer Preference Matching **

## **Overview**
In this task, I have matched **supplier materials** to **buyer preferences** based on **Grade, Finish, and Thickness**. The goal was to generate recommendations that best fit buyer requirements using SQL techniques

---

## **Steps Taken**

### **1. Data Preparation & Cleaning**
- Started with **two supplier datasets** (`supplier_data1`, `supplier_data2`) and **one buyer dataset** (`buyer_preferences`).
- We cleaned and standardized the data by:
  - Ensuring column names were uniform.
  - Removing missing or inconsistent values.
  - Converting numerical values like `Thickness_mm` and `Width_mm` to the correct format.
  - Standardizing text fields (`Grade`, `Finish`).

Python Script for Doing this: https://colab.research.google.com/drive/17bScazEI6hnHl24zxHBCIxINWePVE0hC?usp=sharing

### **2. Uploading Data to Google Cloud Storage**
- After cleaning, uploaded the datasets as **CSV files** to **Google Cloud Storage (GCS)**.
- The files were stored in a **GCS bucket**, making them easily accessible for BigQuery.

### **3. Creating a Database and Tables in BigQuery**
- Created a **new dataset** in BigQuery:  
  **`vanillasteel-452308.task3`**
- Created tables in this dataset:
  - **`buyer_preferences`**
  - **`supplier_data1`**
  - **`supplier_data2`**
- Loaded the CSV files from **Google Cloud Storage (GCS) into BigQuery tables**.

---

## **4. Generating Supplier Recommendations**
We created multiple recommendation tables in BigQuery based on different matching criteria.

### **4.1 Grade-Based Recommendation**
- Matched suppliers and buyers **only based on Grade**.
- **Table Created**: `recommendations_by_grade`
- **Query Used**:
```sql
CREATE OR REPLACE TABLE `vanillasteel-452308.task3.recommendations_by_grade` AS
WITH supplier_filtered AS (
  SELECT 
    Grade, 
    Finish, 
    Thickness_mm
  FROM `vanillasteel-452308.task3.supplier_data1`
)

SELECT 
  b.Buyer_ID,
  b.Preferred_Grade,
  b.Preferred_Finish,
  b.Thickness_mm AS Required_Thickness_mm,
  s.Grade AS Matched_Grade,
  s.Finish AS Matched_Finish,
  s.Thickness_mm AS Matched_Thickness_mm,
  'Supplier Data 1' AS Source
FROM `vanillasteel-452308.task3.buyer_preferences` AS b
JOIN supplier_filtered AS s
ON b.Preferred_Grade = s.Grade;
```
 **Insight**: This table shows all suppliers offering the same grade materials as buyer preferences.

---

### **4.2 Finish-Based Recommendation**
- Matched suppliers and buyers **only based on Finish**.
- **Table Created**: `recommendations_by_finish`
- **Query Used**:
```sql
CREATE OR REPLACE TABLE `vanillasteel-452308.task3.recommendations_by_finish` AS
WITH supplier_filtered AS (
  SELECT 
    Grade, 
    Finish, 
    Thickness_mm
  FROM `vanillasteel-452308.task3.supplier_data1`
)

SELECT 
  b.Buyer_ID,
  b.Preferred_Grade,
  b.Preferred_Finish,
  b.Thickness_mm AS Required_Thickness_mm,
  s.Grade AS Matched_Grade,
  s.Finish AS Matched_Finish,
  s.Thickness_mm AS Matched_Thickness_mm,
  'Supplier Data 1' AS Source
FROM `vanillasteel-452308.task3.buyer_preferences` AS b
JOIN supplier_filtered AS s
ON b.Preferred_Finish = s.Finish;
```
 **Insight**: Buyers looking for specific surface finishes can find matching suppliers here.

---

### **4.3 Thickness-Based Recommendation**
- Matched suppliers and buyers **only based on Thickness** (allowing ±0.1 mm tolerance).
- **Table Created**: `recommendations_by_thickness`
- **Query Used**:
```sql
CREATE OR REPLACE TABLE `vanillasteel-452308.task3.recommendations_by_thickness` AS
WITH supplier_filtered AS (
  SELECT 
    Grade, 
    Finish, 
    Thickness_mm
  FROM `vanillasteel-452308.task3.supplier_data1`
)

SELECT 
  b.Buyer_ID,
  b.Preferred_Grade,
  b.Preferred_Finish,
  b.Thickness_mm AS Required_Thickness_mm,
  s.Grade AS Matched_Grade,
  s.Finish AS Matched_Finish,
  s.Thickness_mm AS Matched_Thickness_mm,
  'Supplier Data 1' AS Source
FROM `vanillasteel-452308.task3.buyer_preferences` AS b
JOIN supplier_filtered AS s
ON ABS(b.Thickness_mm - s.Thickness_mm) <= 0.1; --allowed ±0.1 mm tolerance as there are no matches
```
**Insight**: This table helps identify suppliers offering materials with similar thickness requirements.

---

### **4.4 Best Recommendation (Considering All Factors)**
- Matched suppliers **based on all three factors: Grade, Finish, and Thickness (±0.1 mm tolerance)**.
- **Table Created**: `best_recommendations`
- **Query Used**:
```sql
CREATE OR REPLACE TABLE `vanillasteel-452308.task3.best_recommendations` AS
WITH supplier_filtered AS (
  SELECT 
    Grade, 
    Finish, 
    Thickness_mm
  FROM `vanillasteel-452308.task3.supplier_data1`
)

SELECT 
  b.Buyer_ID,
  b.Preferred_Grade,
  b.Preferred_Finish,
  b.Thickness_mm AS Required_Thickness_mm,
  s.Grade AS Matched_Grade,
  s.Finish AS Matched_Finish,
  s.Thickness_mm AS Matched_Thickness_mm,
  'Supplier Data 1' AS Source
FROM `vanillasteel-452308.task3.buyer_preferences` AS b
JOIN supplier_filtered AS s
ON b.Preferred_Grade = s.Grade
AND b.Preferred_Finish = s.Finish
AND ABS(b.Thickness_mm - s.Thickness_mm) <= 0.1;
```
**Insight**: This table provides the **most accurate supplier matches** based on all three factors.


---



## **Conclusion**
- The **best_recommendations** table provides the most **precise supplier suggestions**.
- These insights can help **optimize inventory management** and **improve supplier-buyer relationships**.


---

### **How to Access the Tables in BigQuery**
All the Table Generation Queries are saved under the Saved Queries Section and required permission is given. 
The Project Can be searched with ID below for Accessing all the BigQueries
Project ID: vanillasteel-452308

