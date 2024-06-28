# Batch-Data-Import-Workflow-Using-AWS-Step-Functions-and-Amazon-Redshift

**Objective:** Import data from a CSV file stored in an S3 bucket into an Amazon Redshift table.

#### Prerequisites
1. **AWS Account**: Please ensure you have an AWS account with the necessary permissions
2. **AWS CLI**: Installed and configured.
3. **Git**: Installed.
4. **Terraform**: Installed.
5. **Amazon Redshift Cluster**: Set up with necessary tables.
6. **Amazon S3 Bucket**: Store the CSV files.

#### Workflow Steps

1. **Setup Infrastructure**
    - **Clone Repository**:
      ```bash
      git clone https://github.com/aws-samples/step-functions-workflows-collection
      cd step-functions-workflows-collection/distributed-map-csv-iterator-tf
      ```
    - **Initialize Terraform**:
      ```bash
      terraform init
      ```
    - **Apply Terraform Configuration**:
      ```bash
      terraform apply
      ```
      - Confirm with `yes`.

2. **Upload CSV File to S3**
    - Upload your CSV file to the designated S3 bucket.

3. **Modify State Machine**
    - Update the state machine definition to include steps to load data into Amazon Redshift.
    - Example State Machine Definition:
      ```json
      {
        "Comment": "A description of my state machine",
        "StartAt": "ProcessCSV",
        "States": {
          "ProcessCSV": {
            "Type": "Map",
            "ItemProcessor": {
              "ProcessorConfig": {
                "Mode": "DISTRIBUTED",
                "ExecutionType": "CHILD"
              },
              "StartAt": "LoadToRedshift",
              "States": {
                "LoadToRedshift": {
                  "Type": "Task",
                  "Resource": "arn:aws:states:::redshift-data:executeStatement.sync",
                  "Parameters": {
                    "ClusterIdentifier": "YOUR-REDSHIFT-CLUSTER-ID",
                    "Database": "YOUR-REDSHIFT-DB",
                    "Sql": {
                      "Fn::Sub": [
                        "COPY ${TableName} FROM 's3://${BucketName}/${FileKey}' CREDENTIALS 'aws_iam_role=${IamRole}' CSV;",
                        {
                          "TableName": "YOUR-TABLE-NAME",
                          "BucketName": "YOUR-BUCKET-NAME",
                          "FileKey": "PREFIX/metrics.csv",
                          "IamRole": "YOUR-REDSHIFT-IAM-ROLE"
                        }
                      ]
                    }
                  },
                  "End": true
                }
              }
            },
            "End": true
          }
        }
      }
      ```

4. **Trigger State Machine**
    - **Start Execution**:
      - Use the following input:
        ```json
        {
          "BucketName": "YOUR-BUCKET-NAME",
          "FileKey": "PREFIX/metrics.csv"
        }
        ```

5. **Verify Data in Redshift**
    - Connect to your Redshift cluster and query the table to verify the data has been loaded.

6. **Cleanup Resources**
    - **Destroy Resources**:
      ```bash
      terraform destroy
      ```
      - Confirm with `yes`.
