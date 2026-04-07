# Data_Engineering_Project_Biz
END TO END PIPLINE FOR E-COMMERCE  DATA


#  E-Commerce Data Pipeline: My End-to-End Project

Hi! In this project, I built a complete system that takes raw sales data and turns it into clean, useful reports. I used the "Medallion Architecture" (Bronze, Silver, Gold) on Azure. Here is exactly how I did it:

### **Step 1: Setting up the "Front Door" (SFTP & FileZilla)**
Instead of just uploading files, I wanted to mimic a real business scenario where a vendor sends us data.
* **Azure Setup**: I enabled the **SFTP feature** on my Azure Storage account.
* **Access**: I created a **local user** and set up an **SSH password**. I gave this user access to a specific container and a landing folder I named `sftp`.
* **FileZilla**: I used the FileZilla client to connect. I entered my **Host, Port, Username, and Password**, and then I dropped my raw files (`cart.json` and `products.json`) into that `sftp` directory. 

### **Step 2: Moving the Data (Azure Data Factory)**
Once the files were in the `sftp` folder, I needed to move them into my data lake.
* I used a **Copy Data Activity** in Azure Data Factory (ADF).
* This activity "grabs" the files from the SFTP folder and "pushes" them into my **Bronze** storage area.
* I also set up a **Storage Event Trigger**. The second I drop a file in SFTP, the whole pipeline wakes up and starts working!

### **Step 3: The Processing Brains (Databricks Notebooks)**
I wrote 6 notebooks to handle the data. You can find them in this repo:

1.  **`01_External_Location`**: I connected Databricks to my Azure storage using Unity Catalog. No passwords—just secure, managed identities.
2.  **`02_Catalog_Schema_Setup`**: I built the "Filing System." I created a Catalog and a Schema so all my tables have a clean, organized home.
3.  **`03_Bronze_Setup`**: I took the raw JSON files and flattened them out. Since one cart can have many products, I used the `explode` function to give every item its own row. I used a "Merge" (Upsert) for carts so I don't get duplicates.
4.  **`04_Silver_Transform`**: This is the cleaning stage. I changed prices from text to proper decimals and handled any missing (NULL) values. I also made sure that if a product's price changes, we remember the old price (SCD Type 2).
5.  **`Gold_Layer`**: The final math! I joined my tables to calculate **Total Revenue** and **Top Selling Products** etc..

### **Step 4: Serving & Alerts**
* **Azure SQL**: I pushed the final Gold tables into an **Azure SQL Database** using a JDBC connection. This makes the data ready for a manager to see in a dashboard.
* **Logic Apps (Email)**: I didn't want to keep checking if the pipeline worked. I set up a **Logic App** and a **Web Activity**. If the pipeline succeeds, I get a "Success" email. If it fails, I get an "Error" email immediately.
* **Storage Event Trigger**: I set up a "motion sensor" for my data—the second a new file lands in the folder(named incremental), the entire pipeline wakes up and runs itself automatically.
* **Concurrency Control**: I limited the system to only process one file at a time. This acts like a "queue" to make sure different runs don't crash into each other or cause errors while saving the final results to the database.

---

###  Project Proof **

* **Screenshot 1**: My SFTP folder in Azure showing the uploaded files.
  <img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/b2619ec6-5b39-494f-b227-23b28a421dee" />

  <img width="1056" height="414" alt="image" src="https://github.com/user-attachments/assets/e6512336-a4e8-452f-a88d-88e04fa0fdb1" />
  

* **Screenshot 2**: My ADF Pipeline showing the green "Succeeded" checkmarks.

  <img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/71aac9ac-dbcf-437d-871d-bc231f73ea44" />

  <img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/8377e1a2-34c7-4edc-836c-808a82df5e35" />

  <img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/5e72c384-d5de-448a-b742-c42f5103bc34" />

  <img width="1360" height="494" alt="image" src="https://github.com/user-attachments/assets/383fd954-6f3f-4124-8f75-26f3c0083b19" />


  

  
* **Screenshot 3**: My "Success" email notification in my inbox.
  
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/765a664a-c9ed-4fa6-a2b2-d2020fe8bbb3" />


### **What I learned**
The biggest lesson was that "Data Engineering" isn't just writing code it's about making sure everything connects securely. I learned how to handle "Race Conditions" by setting **Concurrency to 1** and how to keep my system "Idempotent" so I never double-count a sale.

---

### **Tools I Used:**
* **Azure Storage (ADLS Gen2)**
* **Azure Data Factory (ADF)**
* **Azure Databricks (Spark)**
* **Azure SQL Database**
* **Logic Apps**
* **FileZilla**

