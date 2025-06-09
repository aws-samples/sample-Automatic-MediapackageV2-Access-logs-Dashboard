# AWS MediaPackage v2 Access Logs Dashboard - AMALD v2

This automated solution helps media teams gain valuable insights into traffic patterns and requests reaching their AWS MediaPackage v2 Channel Group deployment. The dashboard provides comprehensive visualization of both ingress and egress access logs, making it an essential tool for monitoring performance and troubleshooting issues.

## Dashboard Preview

### MediaPackage v2 Ingress Logs Dashboard

![MediaPackage v2 Ingress Dashboard - 1](/images/ingress/mpv2_ingress_1.png)

See more examples in [`images/ingress`](/images/ingress/)

### MediaPackage v2 Egress Logs Dashboard

![MediaPackage v2 Egress Dashboard - 2](/images/egress/mpv2_egress_2.png)

See more examples in [`images/egress`](/images/egress/)

## Architecture Design 

![Architecture Diagram](/images/architecture.jpg)

The solution follows this data flow:

1. AWS MediaPackage v2 access logs are ingested through an Amazon CloudWatch Logs Delivery Source (new integration).
2. Logs are then sent through an Amazon CloudWatch Logs Delivery Source to Amazon Data Firehose.
3. Data transformations are applied to the incoming log records by Data Firehose using a Lambda function. The buffered output is then inserted to Apache Iceberg tables stored in an existing S3 bucket.
4. Athena is used to query Apache Iceberg tables from S3, leveraging the tables schema stored in AWS Glue Data Catalog. 
5. Amazon QuickSight is then plugged to Athena as a Data Source to visualize the MediaPackage v2 access logs.

## Deployment Instructions

> **Important:** The S3 bucket where MediaPackage v2 access logs are stored must be in the same region as Athena to avoid additional costs related to inter-region data transfer.
 
### Step 1: Deploy AWS MediaPackage v2 access logs Data Pipeline

1. Sign in to the **AWS Management Console**
2. Navigate to **AWS CloudFormation** > **Create Stack** > **With new resources**
3. Download the AWS CloudFormation template [`cloudformation/mediapackage_v2_logs_analysis.yaml`](./cloudformation/mediapackage_v2_logs_analysis.yaml) from this repository
4. In the CloudFormation create wizard, upload the template file
5. For the **MPV2ChannelGroupArn** parameter, specify the ARN of your MediaPackage v2 Channel Group for which you'd like to analyze logs.
5. For the **`S3DestinationBucket`** parameter, enter the name of an existing S3 bucket where you want to store AWS MediaPackage v2 access logs.
6. Specify a **Stack name** and choose **Next**
7. Leave the **Configure stack options** at default values and choose **Next**
8. Review the details and under **Capabilities**, select the checkbox for **"I acknowledge that AWS CloudFormation might create IAM resources with custom names"**
9. Choose **Submit**

After a successful stack creation, the following resources will be deployed:
- Amazon CloudWatch Logs Delivery Source to Amazon Data Firehose (2)
- AWS Glue Database (1)
- AWS Glue Tables (2)
- Amazon Data Firehose delivery stream (1)
- Amazon Data Firehose Log Group (2) and Log Stream (2)
- AWS Lambda function for data transformation (1)
- Required IAM Policies and roles (2)

### Step 2: Set Up Amazon QuickSight

Amazon QuickSight is AWS's Business Intelligence tool that allows you to visualize your MediaPackage v2 access logs data. If you're already using QuickSight, you can skip to Step 3.

1. Log into your AWS Account and search for QuickSight in the list of Services
2. Sign up for QuickSight if you haven't already
3. When prompted, select the **Enterprise Edition**
4. Select **Continue** and complete the account creation process:
   - Choose the AWS region where your S3 bucket containing MediaPackage access logs is located
5. In the QuickSight permissions section, enable access to **Amazon S3** and select the bucket where your MediaPackage access logs are stored

### Step 3: Deploy the Dashboards

We'll use the provided YAML templates to deploy both the Ingress and Egress access logs dashboards in QuickSight.

#### Install the CID-CMD Tool

```bash
pip3 install cid-cmd
```

#### Deploy the Ingress Logs Dashboard

1. Open your terminal and navigate to the directory containing the template files
2. Run the following command to deploy the [ingress logs dashboard template](./dashboards/mediapackage_v2_ingress_logs.yaml):

```bash
cid-cmd deploy --resources ./dashboards/mediapackage_v2_ingress_logs.yaml
```

3. When prompted, provide the following information:
   - For `[dashboard-id]`, select `mediapackage-v2-logs-ingress-analysis`
   - For `[athena-database]`, press **Enter** to select `mpv2_logs_db`
   - For `[s3path]`, enter the S3 URI where your Ingress MediaPackage v2 access logs are stored (e.g., `s3://your-bucket-name/mediapackage-v2-processed-logs/ingress/`)

Upon successful deployment, you'll see a confirmation message with a link to access your dashboard.

#### Deploy the Egress Logs Dashboard

1. In the same terminal, run the following command to deploy the [egress logs dashboard template](./dashboards/mediapackage_v2_egress_logs.yaml):

```bash
cid-cmd deploy --resources ./dashboards/mediapackage_v2_egress_logs.yaml
```

2. When prompted, provide the following information:
   - For `[dashboard-id]`, select `mediapackage-v2-logs-egress-analysis`
   - For `[athena-database]`, press **Enter** to select `mpv2_logs_db`
   - For `[s3path]`, enter the S3 URI where your Egress MediaPackage v2 access logs are stored (e.g., `s3://your-bucket-name/mediapackage-v2-processed-logs/egress/`)

Upon successful deployment, you'll see a confirmation message with a link to access your dashboard.

## Using the Dashboards

Once deployed, you can access your dashboards through the QuickSight console or via the direct links provided in the deployment output. The dashboards provide:

- Request volume analysis
- HTTP status code distribution
- Geographic distribution of requests
- Client device and browser analytics
- Error rate monitoring
- Performance metrics

## Troubleshooting

If you encounter issues during deployment:

1. Verify that your S3 bucket exists and is in the same region as Athena
2. Check that you have the necessary permissions to create CloudFormation stacks and QuickSight resources

## Contributing

Contributions to improve this solution are welcome. Please feel free to submit pull requests or open issues to suggest enhancements.

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the  [LICENSE](LICENSE) file.