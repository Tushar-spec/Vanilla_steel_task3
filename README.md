# Pipeline for Matching Suppliers with Buyer Preferences

## **Task 3**
### **Objective:** Match supplier materials to buyer preferences and create a recommendation table.
- **Tools:** Use a desired database of your choice.
- **Task:** Create a data pipeline that:
  - Joins the supplier data with buyer preferences.
  - Identifies materials that match buyer preferences based on criteria like grade, finish, and thickness.
  - Outputs a table that lists recommended materials for each buyer.

---

## 📌 **Overview of the Pipeline and GCP Services Used**
This project automates the process of matching **supplier materials** with **buyer preferences** using **Google Cloud Dataflow** and **BigQuery**.

✔ Reads supplier and buyer data from **Google Cloud Storage (GCS)**  
✔ Matches supplier materials to buyer preferences  
✔ Stores the recommendations in **BigQuery** dataset as `recommendations`.  

---

## 🚀 **Prerequisites to Make This Pipeline**
### **1️⃣ Set Up Google Cloud Environment**
Ensure you have:
- **Google Cloud SDK Installed**: Use the command below to install it. This will provide a separate link on Google Cloud Shell. Ensure that you have enabled APIs for Google Cloud Shell.
```sh
gcloud services enable dataflow.googleapis.com bigquery.googleapis.com storage.googleapis.com
gcloud auth login
```
- **GCP Project:** Check the project name `vanilla-steel-task-2`

### **2️⃣ Enable Required GCP Services** using the above command.

---

## 🔄 **Step 1: Upload Files to Google Cloud Storage (GCS)**
The necessary files have already been uploaded to the bucket.

📸 **Snapshot:**  
![Files in GCS](https://github.com/user-attachments/assets/fd4fabfc-9618-4c21-8505-89c862d066ba)

### **1️⃣ Verify Again If Files Exist** (Run this in Cloud Shell)
```sh
gsutil ls gs://vanila_steel_task_2/resources/task_3/
```
✔ Expected output:
```
gs://vanila_steel_task_2/resources/task_3/buyer_preferences.csv
gs://vanila_steel_task_2/resources/task_3/supplier_data1.csv
gs://vanila_steel_task_2/resources/task_3/supplier_data2.csv
```

---

## 🛠 **Step 2: Create BigQuery Dataset & Table**
### **1️⃣ Create a BigQuery Dataset**
```sh
bq mk --location=US vanila_steel_dataset_1
```

### **2️⃣ Create a BigQuery Table** (Check data types before proceeding)
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

### Ingestion Script in Python
The Python script `ingestion.py` developed in Visual Studio Code is crucial for this pipeline. It processes and writes data to BigQuery.

```
import apache_beam as beam
from apache_beam.options.pipeline_options import PipelineOptions, GoogleCloudOptions, SetupOptions
import csv
from io import StringIO
import logging

logging.basicConfig(level=logging.INFO, format='%(asctime)s: %(levelname)s: %(message)s')

PROJECT_ID = 'vanilla-steel-task-2'
BUCKET_NAME = 'vanila_steel_task_2'
DATASET_ID = 'vanila_steel_dataset_1'
TABLE_ID = 'recommendations'

def parse_csv(line):
    try:
        reader = csv.DictReader(StringIO(line), delimiter=',')
        return next(reader)
    except Exception as e:
        logging.error(f'Failed to parse CSV line: {line} - Error: {e}')
        return None

def run(argv=None):
    pipeline_options = PipelineOptions(argv)
    google_cloud_options = pipeline_options.view_as(GoogleCloudOptions)
    google_cloud_options.project = PROJECT_ID
    google_cloud_options.job_name = 'ingestion-job'
    google_cloud_options.region = 'us-central1'
    google_cloud_options.staging_location = f'gs://{BUCKET_NAME}/staging'
    google_cloud_options.temp_location = f'gs://{BUCKET_NAME}/temp'
    pipeline_options.view_as(SetupOptions).save_main_session = True

    with beam.Pipeline(options=pipeline_options) as p:
        raw_data = (p
                    | 'Read CSV File' >> beam.io.ReadFromText(f'gs://{BUCKET_NAME}/resources/task_3/data.csv')
                    | 'Parse CSV Lines' >> beam.Map(parse_csv)
                    | 'Filter None Values' >> beam.Filter(lambda x: x is not None))

        raw_data | 'Write to BigQuery' >> beam.io.WriteToBigQuery(
            f'{PROJECT_ID}:{DATASET_ID}.{TABLE_ID}',
            schema='SCHEMA_AUTODETECT',
            create_disposition=beam.io.BigQueryDisposition.CREATE_IF_NEEDED,
            write_disposition=beam.io.BigQueryDisposition.WRITE_APPEND
        )

if __name__ == '__main__':
    run()
```

Authenticate your project on Google Cloud Shell and run the script using `python ingestion.py`. The entire script is also provided in the resources repository. Once running, check the job in Cloud Dataflow and inspect the dataset in BigQuery.
