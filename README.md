# Google Cloud Dataflow Pipeline for Matching Suppliers with Buyer Preferences

Task 3 
 Objective: Match supplier materials to buyer preferences and create a recommendation table. 
● Tools: Use a desired database of your choice. 
● Task: Create a data pipeline that: 
○ Joins the supplier data with buyer preferences. 
○ Identifies materials that match buyer preferences based on criteria like grade, finish, and thickness. 
○ Outputs a table that lists recommended materials for each buyer. 


## 📌 Overview of the pipeline and GCP services used
This project automates the process of matching **supplier materials** with **buyer preferences** using **Google Cloud Dataflow** and **BigQuery**.

👉 what it does is Reads supplier and buyer data from **Google Cloud Storage (GCS)**  
👉 Matches supplier materials to buyer preferences  
👉 Stores the recommendations in **BigQuery** dataset as recommendations. 

---

## 🚀 Prerequisites to make this pipeline
### **1️⃣ Set Up Google Cloud Environment**
Ensure you have:
- **Google Cloud SDK Installed**:
Use the below code to install it, this will provide you a seperate link on google cloud shell, make sure that you have enabled API for google cloud shell.

`gcloud services enable dataflow.googleapis.com bigquery.googleapis.com storage.googleapis.com`
 `gcloud auth login`
 
- **GCP Project:** check the project name `vanilla-steel-task-2`


### **2️⃣ Enable Required GCP Services** from the above mentioned code


---
---

## 🔄 Step 1: Upload Files to Google Cloud Storage (GCS) important step.

I have already added the files which are necessary to build the pipeline.

Snipshot - 
<img width="1440" alt="image" src="https://github.com/user-attachments/assets/fd4fabfc-9618-4c21-8505-89c862d066ba" />

### **1️⃣ Verify again If Files Exist** do this in command shell.
```sh
gsutil ls gs://vanila_steel_task_2/resources/task_3/
```
👉 You should see:
```
gs://vanila_steel_task_2/resources/task_3/buyer_preferences.csv
gs://vanila_steel_task_2/resources/task_3/supplier_data1.csv
gs://vanila_steel_task_2/resources/task_3/supplier_data2.csv
```
---

## 🛠 Step 2: Now make a BigQuery Dataset & Table in the command shell of gcp
### **1️⃣ Create a BigQuery Dataset**
```sh
bq mk --location=US vanila_steel_dataset_1
```

### **2️⃣ Create a BigQuery Table** check the data types
```sh
bq query --use_legacy_sql=false \
'CREATE TABLE vanila_steel_dataset_1.recommendations (
   buyer_id STRING,
   supplier_id STRING,
   material_type STRING,
   price FLOAT64,
   availability STRING
);'
```

---

## 💟 Step 3: Download & Run `ingestion.py` in Cloud Shell
### **1️⃣ Download the Script from GCS**
```sh
gsutil cp gs://vanila_steel_task_2/resources/task_3/ingestion.py ~/
```

### **2️⃣ Verify the File Exists**
```sh
ls -l ~ | grep ingestion.py
```
👉 You should see:
```
-rw-r--r--  1 user cloud-users  12345 Mar 01 14:00 ingestion.py
```

---

## ⚙ Step 4: Install Required Python Libraries
```sh
pip3 install apache-beam pandas google-cloud-storage google-cloud-bigquery
```

---

## 🔑 Step 5: Authenticate & Set Up GCP
```sh
gcloud auth application-default login
gcloud config set project vanilla-steel-task-2
```

---

## 🚀 Step 6: Run `ingestion.py` (Dataflow Pipeline)
```sh
python3 ingestion.py
```

---

## 📊 Step 7: Verify Data in BigQuery
```sh
bq query --use_legacy_sql=false \
'SELECT * FROM vanila_steel_dataset_1.recommendations LIMIT 10;'
```

---

## 📱 Step 8: Monitor & Debug (If Needed)
```sh
gcloud dataflow jobs list  # Check running jobs
gcloud logging read "resource.type=dataflow_step" --limit 50  # Check logs
bq show --format=prettyjson vanila_steel_dataset_1.recommendations  # Verify schema
```

---

## 🛢 Step 9: Cleanup (Optional)
```sh
gsutil rm -r gs://vanila_steel_task_2/resources/task_3/temp/
gcloud dataflow jobs list  # Find the Job ID
gcloud dataflow jobs cancel JOB_ID
```

---

## ✅ Final Summary
✔ **Uploaded CSV files to `gs://vanila_steel_task_2/resources/task_3/`**  
✔ **Created BigQuery dataset (`vanila_steel_dataset_1`) and table (`recommendations`)**  
✔ **Downloaded `ingestion.py` and installed dependencies**  
✔ **Authenticated Google Cloud & set the correct project**  
✔ **Ran the Python script (`ingestion.py`)**  
✔ **Verified data in BigQuery (`vanila_steel_dataset_1.recommendations`)**  
✔ **Monitored logs & debugged errors**  
✔ **Cleaned up temporary files (if needed)**  

🚀 **Now your Dataflow pipeline is fully automated on GCP!** 🎉  
Let me know if you need help! 😊

