# Google Cloud Dataflow Pipeline for Matching Suppliers with Buyer Preferences

## 📌 Overview
This project automates the process of matching **supplier materials** with **buyer preferences** using **Google Cloud Dataflow** and **BigQuery**.

👉 Reads supplier and buyer data from **Google Cloud Storage (GCS)**  
👉 Matches supplier materials to buyer preferences  
👉 Stores the recommendations in **BigQuery**  

---

## 🚀 Prerequisites
### **1️⃣ Set Up Google Cloud Environment**
Ensure you have:
- **Google Cloud SDK Installed**: [Install Here](https://cloud.google.com/sdk/docs/install)
- **GCP Project:** `vanilla-steel-task-2`
- **Billing Enabled** for Dataflow & BigQuery

### **2️⃣ Enable Required GCP Services**
```sh
gcloud services enable dataflow.googleapis.com bigquery.googleapis.com storage.googleapis.com
```

---

## 📂 File Structure
```
/gcp_dataflow_project
👉 ingestion.py         # Apache Beam Dataflow Pipeline
👉 buyer_preferences.csv # Buyer dataset
👉 supplier_data1.csv    # Supplier dataset 1
👉 supplier_data2.csv    # Supplier dataset 2
👉 README.md             # Project documentation
```

---

## 🔄 Step 1: Upload Files to Google Cloud Storage (GCS)
### **1️⃣ Verify If Files Exist**
```sh
gsutil ls gs://vanila_steel_task_2/resources/task_3/
```
👉 You should see:
```
gs://vanila_steel_task_2/resources/task_3/buyer_preferences.csv
gs://vanila_steel_task_2/resources/task_3/supplier_data1.csv
gs://vanila_steel_task_2/resources/task_3/supplier_data2.csv
```

### **2️⃣ Upload Files (If Not Already Uploaded)**
```sh
gsutil cp buyer_preferences.csv gs://vanila_steel_task_2/resources/task_3/
gsutil cp supplier_data1.csv gs://vanila_steel_task_2/resources/task_3/
gsutil cp supplier_data2.csv gs://vanila_steel_task_2/resources/task_3/
```

---

## 🛠 Step 2: Setup BigQuery Dataset & Table
### **1️⃣ Create a BigQuery Dataset**
```sh
bq mk --location=US vanila_steel_dataset_1
```

### **2️⃣ Create a BigQuery Table**
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

