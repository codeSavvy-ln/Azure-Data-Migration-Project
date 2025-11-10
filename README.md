# Azure-Data-Migration-Project
Data is migrated from SQL Server Management Service into Azure SQL Database

---

## **Project Overview**  
This project demonstrates an end-to-end data engineering solution to migrate the data from On-Prem database (SSMS) to Azure SQL DB in increamental manner instead to Full load.  

## **Project Architecture**  
![logo](https://github.com/codeSavvy-ln/Azure-Data-Migration-Project/blob/main/Azure%20Data%20Migration%20Project.png)

---

## **Technologies Used**
- **Programming Language:** SQL
- **Azure Data Factory**
- **Data Sources:**
    - **Azure SQL Database**
    - **Azure storage:** ADLS Gen2, Stored Procedure.
    - **On-Prem Server:** SSMS (SQL Server Management Studio).

---
## **Pipeline Architecture (ETL Pipeline Breakdown)**  

### **Step 1: Load the increamental data into Staging Table** ⚙️

- Fetch the max datetime from target table (Employee1) in our Azure SQL database using the **dataflow activity** and insert it into ADLS Gen2 container in the form of csv.
- Retreives this MaxTimeStamp date from ADLS Gen2 Container using **Lookup activity** which will be used in **Copy data activity**.
- Copy only the data whose time_stamp inside the SSMS Employee1 table is greater than the time_stamp fetched from previous lookup table into the **Staging Table** (Employee1_Staging) inside Azure SQL Database.

### **Step 2: Merge into the target table skipping the duplicate records** 🚀

- Use **Stored Procedure activity** for Upsert query which also skips the duplicate records
```
CREATE PROCEDURE [dbo].[sp_merge_employee1_data]
AS
with cte as ( select *, row_number() over(partition by EmployeeId order by time_stamp desc) as rnk
              from [dbo].[Employee1_Stage])

        MERGE INTO [dbo].[Employee1] AS target
        USING (select * from cte where rnk = 1) AS source
            ON target.EmployeeId = source.EmployeeId

        WHEN MATCHED 
             AND source.time_stamp <> target.time_stamp
        THEN 
            UPDATE SET 
                target.Name = source.Name,
                target.Gender = source.Gender,
                target.Salary = source.Salary,
                target.Department = source.Department,
                target.Experience = source.Experience,
                target.time_stamp = source.time_stamp

        WHEN NOT MATCHED BY TARGET 
        THEN 
            INSERT (EmployeeId, Name, Gender, Salary, Department, Experience, time_stamp)
            VALUES (source.EmployeeId, source.Name, source.Gender, source.Salary, source.Department, source.Experience, source.time_stamp);

```
### **Step 3: Truncate the Staging Table** ⚙️
- Truncate the **Staging Table (Employee1_Staging) for the next use.


## **How to Use**  

### **1. Prerequisites**  
- Active Azure Subscription.  
- Access to SSMS
- Install the Self-hosted integration runtime  

### **2. Steps to Set Up**  
1. Clone this repository:  
   ```bash
   git clone https://github.com/codeSavvy-ln/Azure-Data-Migration-Project.git
