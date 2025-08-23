🌱 Health Metrics Tracker (AWS Event-Driven Pipeline)
*A personal serverless workflow to collect, process, and visualize health data from Garmin, Withings, and MyFitnessPal*

***

🎯 Project Overview  
This repo deploys an event-driven AWS architecture that pulls health metrics daily from your favorite APIs (Garmin for steps/activity, Withings for weight, MyFitnessPal for food). Raw data lands in S3, gets converted to analytics-ready format by Glue, triggers dashboards in Quicksight, and sends you an SNS-powered daily summary email. All orchestration is fully serverless and automated via CloudTrail and EventBridge, following AWS best practices for event-driven workflows.

***

🛠 AWS Services Used

- **API Gateway**: Exposes Lambda endpoints for fetching data from the 3 external APIs.
- **Lambda**: Functions to pull your daily activity, weight, and food logs and store as JSON in S3.
- **S3**: Storage for each raw and processed daily health metric.
- **CloudTrail**: Captures S3 PutObject events for triggering downstream workflows.
- **EventBridge**: Batches events, triggers Glue workflow for ETL when all data for the day is in.
- **Glue Workflow**: Crawlers and jobs transform JSON files to Parquet, update the Data Catalog, and prep for analytics.
- **Athena**: Enables SQL queries for ad-hoc analysis.
- **Quicksight**: Visualizes your health history in clean dashboards.
- **SNS**: Delivers a daily email summarizing yesterday’s metrics.

***

🔄 Architecture Components

**Data Ingestion (APIs + Lambda)**
- `GET /garmin/metrics` (API Gateway → Lambda): Fetches step/activity data.
- `GET /withings/weight` (API Gateway → Lambda): Gets weight records.
- `GET /myfitnesspal/food` (API Gateway → Lambda): Retrieves food diary.

> Example Python snippet: Garmin  
```python
import requests
GARMIN_API_URL = "https://api.garmin.com/wellness-api/rest/activity"
access_token = "YOUR_TOKEN"
headers = {"Authorization": f"Bearer {access_token}"}
resp = requests.get(GARMIN_API_URL, headers=headers)
data = resp.json()  # Save to S3
```

> Example Python snippet: Withings  
```python
import requests
WITHINGS_API_URL = "https://wbsapi.withings.net/measure"
params = {"action": "getmeas", "access_token": "YOUR_TOKEN"}
resp = requests.get(WITHINGS_API_URL, params=params)
data = resp.json()  # Save to S3
```

> Example Python snippet: MyFitnessPal  
```python
import requests
MFP_API_URL = "https://api.myfitnesspal.com/v2/diary"
headers = {"Authorization": "Bearer YOUR_TOKEN"}
resp = requests.get(MFP_API_URL, headers=headers)
data = resp.json()  # Save to S3
```

**S3 Buckets**
- `health-metrics-raw/YYYY-MM-DD/garmin.json`
- `health-metrics-raw/YYYY-MM-DD/withings.json`
- `health-metrics-raw/YYYY-MM-DD/myfitnesspal.json`

**Event Workflow**
1. **Upload triggers CloudTrail events** captured on S3.
2. **EventBridge** batches 3 events (or a set time) before firing Glue workflow.
3. **Glue pre-job** cleans and validates JSON.
4. **Glue ETL job** transforms and merges all daily metrics to Parquet.
5. **Glue Crawler** updates Data Catalog so Athena & Quicksight can query.
6. **Final Glue action** sends summary data to SNS → daily email 📬.

**Visualization & Notification**
- **Athena** queries across days for trend/wellness analysis.
- **Quicksight** delivers interactive dashboards for web/mobile.
- **SNS** email at 7am daily summarizes previous day’s steps, calories, and weight.

***

📝 Implementation Details

1. **API Auth Setup**
   - Set up OAuth tokens for Garmin, Withings, and MyFitnessPal.
   - Secure Lambda code with Secrets Manager or Environment variables.

2. **Lambda & API Gateway Integration**
   - Each API gets its own Lambda function & Gateway resource.
   - Use scheduled EventBridge rules or webhooks for automation.

3. **S3-Bucket Naming and Storage**
   - Organize with `/YYYY-MM-DD/source.json` layout for easy partitioning.

4. **Event Orchestration**
   - CloudTrail tracks uploads, EventBridge batches, Glue is triggered once all/most data is available.

5. **Glue ETL & Catalog**
   - Amazon Glue jobs transform day’s JSON into Parquet partitioned by date.
   - Crawler updates schema for Athena & Quicksight.

6. **Summary Email with SNS**
   - Lambda or Glue sends message to SNS topic after each ETL run.
   - SNS delivers daily wellness email.

***

🔍 Features

- Collects, transforms, and visualizes personal health data 🤸‍♂️
- Automated daily summary in your inbox ✉️
- Scalable, event-driven, and fully serverless (pay-as-you-go)
- Athena & Quicksight powered ad-hoc and historical analytics dashboards
- Easily extensible for more APIs or custom metrics

***

🌟 Future Enhancements

- Expand data sources: add Fitbit, Apple Health, Strava etc.
- Integrate push notifications or Slack alerts for goal achievement.
- Add anomaly detection or health trends analysis (Machine Learning).
- Enable web/mobile personal dashboard.
- Multi-user support for family or friends.

***

🔒 Security Measures

- API Gateway with authentication/authorization checks
- IAM roles scoped to minimum required permissions
- Encrypt S3 buckets
- Audit trail via CloudTrail and AWS Config

***

📊 Performance Metrics

- ETL latency < 5 min per daily batch
- Email summaries delivered by 7:00am daily
- Quicksight dashboards update within 15 min of ingestion

***

💻 Technical Requirements

- AWS Account
- OAuth2 tokens for Garmin, Withings, MyFitnessPal
- Quicksight subscription for dashboards
- SNS email subscription

***

⚠️ Limitations

- API rate limits and token refresh restrictions
- Some APIs (e.g., MyFitnessPal) may change without notice
- S3 bucket region and Glue quotas apply
- Quicksight visualization refresh intervals

***

🚀 *Built with AWS best practices and scalability in mind*  
Feel free to contribute or suggest improvements!

***

#### Reference

☁️ Parts of this architecture and workflow are based on AWS Big Data Blog:  
[Build a serverless event-driven workflow with AWS Glue and Amazon EventBridge](https://aws.amazon.com/blogs/big-data/build-a-serverless-event-driven-workflow-with-aws-glue-and-amazon-eventbridge/)

***

Ready for CloudFormation scripting next?

[1] https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72413388/20ef70be-b418-448d-bbde-28f1ef85881d/Build-a-serverless-event-driven-workflow-with-AWS-Glue-and-Amazon-EventBridge-_-AWS-Big-Data-Blog.pdf
[2] https://aws.amazon.com/blogs/big-data/build-a-serverless-ev
