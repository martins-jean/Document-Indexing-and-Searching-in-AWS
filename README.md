# Document indexing and searching in AWS

## Contextual overview

<p align="justify">
A bank would like to increase the number of online transactions customers can review from 6 months to 5 years. In addition, the online bank statements must support textual searches for all fields in the statement.
</p>

## Architecture diagram

![Screenshot 2023-09-01 at 08 05 36](https://github.com/martins-jean/Document-Indexing-and-Searching-in-AWS/assets/118685801/80438bf1-262e-43c0-a91c-cb471b8ff669)

## Project objectives

<p align="justify">
1. To upload files from on-premises applications and store them in the cloud, we will use S3. <br>
2. To index bank transactions, we will deploy an OpenSearch domain. <br>
3. To read data from S3 and load it into OpenSearch for full-text search capabilities, we will use a Glue ETL job. <br>
4. To explore the data and create queries, we will use the built-in dashboards in OpenSearch.
</p>

## Reproducibility guidelines

<details>
  <summary>Required setup</summary>
  1. Download the "glue_to_opensearch_job.py" file locally <br>
  2. Create an ingestion bucket in S3, make sure it contains the "elasticsearch-hadoop-7.8.0.jar" file <br>
  3. In the S3 bucket, create an "input/" folder and make sure it contains the "transactions.csv.gz" file <br>
  4. Create an IAM role for AWS Glue named "NewGlueServiceRole" with permissions to access S3 for any sources, targets, scripts and temporary directories <br>
  5. Make sure you create a t3.micro EC2 instance called "SearchInstance" <br>
</details>

<details>
  <summary>Deloy and configure an Amazon OpenSearch domain</summary>
  1. Navigate to the OpenSearch console and click on "create domain", using the following configurations: <br>
  - Domain name: bank-transactions <br>
  - Domain creation method: standard create <br>
  - Templates: dev/test <br>
  - Deployment options: domain without standby <br>
  - Availability zones: 1 AZ <br>
  - Enginer options / version: 7.10 <br>
  - Data nodes / instance type: m5.large.search <br>
  - Number of nodes: 1 <br>
  - Network: public access <br>
  - Master user: create master user <br>
  - Master username: project-user <br>
  - Master password: ProjectUserD777! <br>
  - Access policy: only use fine-grained access control <br>
  - Click create at the bottom of the page to finish this step <br>
</details>

<details>
  <summary>Creating an ETL job using AWS Glue studio</summary>
  1. Open S3 and click the "elasticsearch-hadoop-7.8.0.jar" checkbox, then click "Copy S3 URI" above it to a local file on your computer <br>
  2. Click the "input/" folder and "Copy S3 URI" at the top right of the page <br>
  3. Navigate to AWS Glue Studio and create an ETL job with the following configurations: <br>
  - Create job: Spark script editor <br>
  - Options: upload and edit an existing script <br>
  - File upload: click and upload the .py file in this repository <br>
  - Click create and name the script "bank-transactions-ingestion-job" <br>
  4. Switch to the "Job Details" tab and select the following: <br>
  - IAM role: NewGlueServiceRole <br>
  - Glue version: 2.0 <br>
  - Job bookmark: disable <br>
  - Number of retries: 0 <br>
  - Under Advanced Properties, libraries / dependent JARs path paste the first S3 URI you copied and click save <br>
</details>

<details>
  <summary>Configure an ETL script to ingest Amazon S3 data into Amazon OpenSearch</summary>
  1. Navigate to the OpenSearch Service to verify the domain is now available <br>
  2. Click on the domain, copy and paste the domain endpoint locally <br>
  3. Go back to your Glue job, under Job Details / Job Parameters click add new parameter: <br>
  - Key: --es_endpoint <br>
  - Value: URL of the endpoint you copied <br>
  4. Add another parameter:
  - Key: --es_user <br>
  - Value: project-user <br>
  5. Add another parameter:
  - Key: --es_pass <br>
  - Value: ProjectUserD777! <br>
  6. Add another parameter: <br>
  - Key: --input_bucket <br>
  - Value: the S3 URI you copied for the ingestion bucket <br>
  7. Click save and run. <br>
  8. Refresh the run details page to check if your run was completed successfully <br>
</details>

<details>
  <summary>Use the OpenSearch dashboards to call the Search API to query data from the OpenSearch domain</summary>
  1. Navigate to the EC2 console and click on instances <br>
  2. Copy the Public IPv4 address of the instance you created earlier and paste it into a new browser tab <br>
  3. Type the word "credit" for example and click search to see the results <br>
  4. Navigate to the OpenSearch console and click on the domain you created <br>
  5. Click on the Kibana URL and enter the credentials you created earlier <br>
  6. Select "explore on my own" <br>
  7. Select private tenant and click confirm <br>
  8. Click "Interact with the Elasticsearch API" <br>
  9. Review the provided query example and click play, this query searches all indexes in your cluster <br>
  10. After GET type: /main-index/_search <br>
  11. Test the query written in the file "test-query.rtf" listed on this repository <br>
  12. Change the query with new keywords related to bank transactions in order to see new results <br>
</details>
