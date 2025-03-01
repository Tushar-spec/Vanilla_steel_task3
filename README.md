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
## building a data pipeline, I have used visual code studio to make a .py file which is ingestion.py, the code is attached below -

import apache_beam as beam
from apache_beam.options.pipeline_options import PipelineOptions
import csv
import io
import pandas as pd
from google.cloud import storage, bigquery

# GCP Configurations
PROJECT_ID = "vanilla-steel-task-2"  # Replace with your actual GCP Project ID
BUCKET_NAME = "vanila_steel_task_2"
DATASET_ID = "vanila_steel_dataset_1"  # Updated dataset name
TABLE_ID = "recommendations"

# GCS file paths
BUYER_PREFERENCES_FILE = f"gs://{BUCKET_NAME}/resources/task_3/buyer_preferences.csv"
SUPPLIER_DATA1_FILE = f"gs://{BUCKET_NAME}/resources/task_3/supplier_data1.csv"
SUPPLIER_DATA2_FILE = f"gs://{BUCKET_NAME}/resources/task_3/supplier_data2.csv"

class ReadCSVFile(beam.DoFn):
    """Reads CSV file from GCS and returns a list of dictionaries."""
    def __init__(self, file_path):
        self.file_path = file_path

    def process(self, element):
        client = storage.Client()
        bucket = client.bucket(BUCKET_NAME)
        blob = bucket.blob(self.file_path.split("/")[-1])  # Extract filename
        content = blob.download_as_text()

        # Read CSV into a list of dictionaries
        reader = csv.DictReader(io.StringIO(content))
        return [row for row in reader]

class MatchSupplierWithBuyer(beam.DoFn):
    """Matches supplier materials with buyer preferences."""
    def process(self, element):
        buyer_data, supplier_data = element

        # Convert list of dicts to DataFrame
        buyer_df = pd.DataFrame(buyer_data)
        supplier_df = pd.DataFrame(supplier_data)

        # Standardize column names
        buyer_df.columns = buyer_df.columns.str.lower().str.replace(" ", "_")
        supplier_df.columns = supplier_df.columns.str.lower().str.replace(" ", "_")

        # Merge supplier datasets
        merged_df = buyer_df.merge(supplier_df, on='material_type', how='inner')

        # Generate recommendation table
        recommendations = merged_df[["buyer_id", "supplier_id", "material_type", "price", "availability"]]

        return recommendations.to_dict(orient="records")

class WriteToBigQuery(beam.DoFn):
    """Writes the output data to BigQuery."""
    def process(self, element):
        client = bigquery.Client()
        table_ref = f"{PROJECT_ID}.{DATASET_ID}.{TABLE_ID}"

        job = client.insert_rows_json(table_ref, [element])
        if job:
            print(f"Error inserting row: {job}")
        return

def run():
    """Runs the Apache Beam pipeline."""
    options = PipelineOptions(
        runner="DataflowRunner",  # Change to 'DirectRunner' for local testing
        project=PROJECT_ID,
        temp_location=f"gs://{BUCKET_NAME}/resources/task_3/temp",
        region="us-central1",
        staging_location=f"gs://{BUCKET_NAME}/resources/task_3/staging"
    )

    with beam.Pipeline(options=options) as p:
        buyer_prefs = p | "Read Buyer Preferences" >> beam.ParDo(ReadCSVFile(BUYER_PREFERENCES_FILE))
        supplier_data1 = p | "Read Supplier Data 1" >> beam.ParDo(ReadCSVFile(SUPPLIER_DATA1_FILE))
        supplier_data2 = p | "Read Supplier Data 2" >> beam.ParDo(ReadCSVFile(SUPPLIER_DATA2_FILE))

        supplier_data = (supplier_data1, supplier_data2) | beam.Flatten()

        recommendations = (
            (buyer_prefs, supplier_data)
            | "Match Suppliers with Buyers" >> beam.ParDo(MatchSupplierWithBuyer())
        )

        recommendations | "Write to BigQuery" >> beam.ParDo(WriteToBigQuery())

if __name__ == "__main__":
    run()

save and upload this file to gcp cloud, and make sure that all the necessary libraries are installed.


## Run `ingestion.py` in Cloud Shell
### **1️⃣ Download the Script from GCP Bucket**
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

The architecture is simple and explained below -

```mermaid
graph TD;
    A[GCS Storage] -->|CSV Files| B[Google Cloud Dataflow];
    B -->|Processed Data| C[BigQuery - recommendations Table];
    C -->|SQL Queries| D[Data Analysis];
    B -->|Logs| E[Cloud Logging];




