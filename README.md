# ITCS-6190 Assignment 3: AWS Data Processing Pipeline

This project demonstrates an end-to-end serverless data processing pipeline on AWS. The process involves ingesting raw data into S3, using a Lambda function to process it, cataloging the data with AWS Glue, and finally, querying and visualizing the results on a dynamic webpage hosted on an EC2 instance.

## 1. Amazon S3 Bucket Structure 🪣

First, set up an S3 bucket with the following folder structure to manage the data workflow:

* **`bucket-name/`**
    * **`raw/`**: For incoming raw data files.
    * **`processed/`**: For cleaned and filtered data output by the Lambda function.
    * **`enriched/`**: For storing athena query results.

---

<img width="648" height="396" alt="image" src="https://github.com/user-attachments/assets/575ba3b6-bac5-4cce-9ebe-42abfd45de2d" />
I created the bucket with the specified file structure.

## 2. IAM Roles and Permissions 🔐

Create the following IAM roles to grant AWS services the necessary permissions to interact with each other securely.

### Lambda Execution Role

1.  Navigate to **IAM** -> **Roles** and click **Create role**.
2.  **Trusted entity type**: Select **AWS service**.
3.  **Use case**: Select **Lambda**.
4.  **Add Permissions**: Attach the following managed policies:
    * `AWSLambdaBasicExecutionRole`
    * `AmazonS3FullAccess`
5.  Give the role a descriptive name (e.g., `Lambda-S3-Processing-Role`) and create it.

### Glue Service Role

1.  Create another IAM role for **AWS service** with the use case **Glue**.
2.  **Add Permissions**: Attach the following policies:
    * `AmazonS3FullAccess`
    * `AWSGlueConsoleFullAccess`
    * `AWSGlueServiceRole`
3.  Name the role (e.g., `Glue-S3-Crawler-Role`) and create it.

### EC2 Instance Profile

1.  Create a final IAM role for **AWS service** with the use case **EC2**.
2.  **Add Permissions**: Attach the following policies:
    * `AmazonS3FullAccess`
    * `AmazonAthenaFullAccess`
3.  Name the role (e.g., `EC2-Athena-Dashboard-Role`) and create it.

---

<img width="1071" height="611" alt="image" src="https://github.com/user-attachments/assets/b0b0c17c-4bd5-4749-9b39-da6ba5ef1dd4" />

<img width="799" height="655" alt="image" src="https://github.com/user-attachments/assets/280179ed-9ee6-4012-bd65-1bdcffa096c9" />

<img width="731" height="605" alt="image" src="https://github.com/user-attachments/assets/0c55c46a-ad7b-41d5-ae04-ee59fdc90e38" />

I created these roles in IAM, with the purpose of assigning them to allow these services to access other services.

## 3. Create the Lambda Function ⚙️

This function will automatically process files uploaded to the `raw/` S3 folder.

1.  Navigate to the **Lambda** service in the AWS Console.
2.  Click **Create function**.
3.  Select **Author from scratch**.
4.  **Function name**: `FilterAndProcessOrders`
5.  **Runtime**: Select **Python 3.9** (or a newer version).
6.  **Permissions**: Expand *Change default execution role*, select **Use an existing role**, and choose the **Lambda Execution Role** you created.
7.  Click **Create function**.
8.  In the **Code source** editor, replace the default code with LambdaFunction.py code for processing the raw data.

---

<img width="1635" height="772" alt="image" src="https://github.com/user-attachments/assets/b1ac578a-6d0c-4696-903d-0eb1d82dc907" />

I created the Lambda function, this will be used to run event-driven code to process new entries in the S3 bucket... speaking of which...

## 4. Configure the S3 Trigger ⚡

Set up the S3 trigger to invoke your Lambda function automatically.

1.  In the Lambda function overview, click **+ Add trigger**.
2.  **Source**: Choose **S3**.
3.  **Bucket**: Select your S3 bucket.
4.  **Event types**: Choose **All object create events**.
5.  **Prefix (Required)**: Enter `raw/`. This ensures the function only triggers for files in this folder.
6.  **Suffix (Recommended)**: Enter `.csv`.
7.  Check the acknowledgment box and click **Add**.

--- 

<img width="706" height="400" alt="image" src="https://github.com/user-attachments/assets/6abde81f-cfc1-4c60-aee3-22e5348264f6" />

I set the trigger, allowing the Lambda function to run.


**Start Processing of Raw Data**: Now upload the Orders.csv file into the `raw/` folder of the S3 Bucket. This will automatically trigger the Lambda function.

<img width="548" height="341" alt="image" src="https://github.com/user-attachments/assets/49502c1c-1541-48da-80dd-2afc5324bf8e" />

Done.

---

## 5. Create a Glue Crawler 🕸️

The crawler will scan your processed data and create a data catalog, making it queryable by Athena.

1.  Navigate to the **AWS Glue** service.
2.  In the left pane, select **Crawlers** and click **Create crawler**.
3.  **Name**: `orders_processed_crawler`.
4.  **Data source**: Point the crawler to the `processed/` folder in your S3 bucket.
5.  **IAM Role**: Select the **Glue Service Role** you created earlier.
6.  **Output**: Click **Add database** and create a new database named `orders_db`.
7.  Finish the setup and run the crawler. It will create a new table in your `orders_db` database.

---

<img width="1038" height="304" alt="image" src="https://github.com/user-attachments/assets/6cb896b2-d3fe-4deb-b98f-0165a84dff66" />

I created the Glue Crawler to crawl processed orders.

<img width="1519" height="80" alt="image" src="https://github.com/user-attachments/assets/7120a112-6e3a-4ed3-9642-2cd0513aecc6" />

I also ran the crawler.


## 6. Query Data with Amazon Athena 🔍

Navigate to the **Athena** service. Ensure your data source is set to `AwsDataCatalog` and the database is `orders_db`. You can now run SQL queries on your processed data.

**Queries to be executed:**
* **Total Sales by Customer**: Calculate the total amount spent by each customer.
* **Monthly Order Volume and Revenue**: Aggregate the number of orders and total revenue per month.
* **Order Status Dashboard**: Summarize orders based on their status (`shipped` vs. `confirmed`).
* **Average Order Value (AOV) per Customer**: Find the average amount spent per order for each customer.
* **Top 10 Largest Orders in February 2025**: Retrieve the highest-value orders from a specific month.

---
<img width="492" height="146" alt="image" src="https://github.com/user-attachments/assets/e710d3bd-d394-4d40-87cc-6c00b1187277" />
<img width="836" height="362" alt="image" src="https://github.com/user-attachments/assets/d5899e5f-cd34-4831-afcc-e51b7169b915" />
Query 1 and result 1.

<img width="1005" height="142" alt="image" src="https://github.com/user-attachments/assets/6ea713df-5daf-4045-8a18-73a74194e34c" />
<img width="1033" height="288" alt="image" src="https://github.com/user-attachments/assets/0e6d5a83-39eb-4efc-b771-38c1f48322fd" />
Query 2 and result 2.

<img width="687" height="111" alt="image" src="https://github.com/user-attachments/assets/9b499584-b0b9-4cca-aba7-037b1152804c" />
<img width="1286" height="200" alt="image" src="https://github.com/user-attachments/assets/6801f7f4-23c5-411b-bebf-31ded59d4678" />
Query 3 and result 3.

<img width="605" height="153" alt="image" src="https://github.com/user-attachments/assets/7a86af28-65fd-444d-a83d-13160d6d37d0" />
<img width="889" height="331" alt="image" src="https://github.com/user-attachments/assets/d088eade-0604-4491-aea2-7fc377929817" />
Query 4 and result 4.

<img width="758" height="149" alt="image" src="https://github.com/user-attachments/assets/894fe9e0-7405-4ab6-9616-38d269566abb" />
<img width="1323" height="517" alt="image" src="https://github.com/user-attachments/assets/7108f80d-3bd5-405f-b9e8-992bad442612" />
Query 5 and result 5.


## 7. Launch the EC2 Web Server 🖥️

This instance will host a simple web page to display the Athena query results.

1.  Navigate to the **EC2** service and click **Launch instance**.
2.  **Name**: `Athena-Dashboard-Server`.
3.  **Application and OS Images**: Select **Amazon Linux 2023 AMI**.
4.  **Instance type**: Choose **t2.micro** (Free tier eligible).
5.  **Key pair (login)**: Create and download a new key pair. **Save the `.pem` file!**
6.  **Network settings**: Click **Edit** and configure the security group:
    * **Rule 1 (SSH)**: Type: `SSH`, Port: `22`, Source: `My IP`.
    * **Rule 2 (Web App)**: Click **Add security group rule**.
        * Type: `Custom TCP`
        * Port Range: `5000`
        * Source: `Anywhere` (`0.0.0.0/0`)
7.  **Advanced details**: Scroll down and for **IAM instance profile**, select the **EC2 Instance Profile** you created.
8.  Click **Launch instance**.

---
<img width="1000" height="217" alt="image" src="https://github.com/user-attachments/assets/9db1bb43-6f5d-4ded-8287-1ee11105e814" />
I launched the EC2 instance using the proper settings.


## 8. Connect to Your EC2 Instance

1.  From the EC2 dashboard, select your instance and copy its **Public IPv4 address**.
2.  Open a terminal or SSH client and connect using your key pair:

    ```bash
    ssh -i /path/to/your-key-file.pem ec2-user@YOUR_PUBLIC_IP_ADDRESS
    ```

---
<img width="1703" height="269" alt="image" src="https://github.com/user-attachments/assets/ab5055e5-a740-4b32-b874-e0822f4fdb92" />
Connected!!


## 9. Set Up the Web Environment

Once connected via SSH, run the following commands to install the necessary software.

1.  **Update system packages**:
    ```bash
    sudo yum update -y
    ```
2.  **Install Python and Pip**:
    ```bash
    sudo yum install python3-pip -y
    ```
3.  **Install Python libraries (Flask & Boto3)**:
    ```bash
    pip3 install Flask boto3
    ```

---
<img width="1903" height="66" alt="image" src="https://github.com/user-attachments/assets/edc95058-ef96-4c73-a7d5-9f201dcd2bfa" />
Everything is set up! (mind the terrible terminal, had to install xfce beause surprise surprise gnome broke)


## 10. Create and Configure the Web Application

1.  Create the application file using the `nano` text editor:
    ```bash
    nano app.py
    ```
2.  Copy and paste your Python web application code (`EC2InstanceNANOapp.py`) into the editor.

3.  ‼️ **Important**: Update the placeholder variables at the top of the script:
    * `AWS_REGION`: Your AWS region (e.g., `us-east-1`).
    * `ATHENA_DATABASE`: The name of your Glue database (e.g., `orders_db`).
    * `S3_OUTPUT_LOCATION`: The S3 URI for your Athena query results (e.g., `s3://your-athena-results-bucket/`).

4.  Save the file and exit `nano` by pressing `Ctrl + X`, then `Y`, then `Enter`.

---
<img width="1810" height="246" alt="image" src="https://github.com/user-attachments/assets/9a1d4e3e-5ee4-46c4-ac62-20c35e789324" />
Done.


## 11. Run the App and View Your Dashboard! 🚀

1.  Execute the Python script to start the web server:
    ```bash
    python3 app.py
    ```
    You should see a message like `* Running on http://0.0.0.0:5000/`.

2.  Open a web browser and navigate to your instance's public IP address on port 5000:
    ```
    http://YOUR_PUBLIC_IP_ADDRESS:5000
    ```
    You should now see your Athena Orders Dashboard!

---
<img width="689" height="83" alt="image" src="https://github.com/user-attachments/assets/45b770b2-0b41-468d-91a1-f2b2e47497f7" />
<img width="1543" height="887" alt="image" src="https://github.com/user-attachments/assets/d8dd6a61-83cf-44aa-ae28-8ff26d2c1465" />
<img width="1565" height="852" alt="image" src="https://github.com/user-attachments/assets/448f740a-3678-4490-ba74-c31b98a5dee0" />
The EC2 instance is running! The dashboard is up and publicly accessible.


## Important Final Notes

* **Stopping the Server**: To stop the Flask application, return to your SSH terminal and press `Ctrl + C`.
* **Cost Management**: This setup uses free-tier services. To prevent unexpected charges, **stop or terminate your EC2 instance** from the AWS console when you are finished.
