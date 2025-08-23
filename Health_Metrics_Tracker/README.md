# 🌱 Health Metrics Tracker (AWS Event-Driven Pipeline)

*A personal, serverless workflow to collect, process, and visualize health data from **Garmin**, **Withings**, and **MyFitnessPal** – all powered by AWS.*

***

## 🎯 Project Overview

This repo lets you automate your health data flow with an AWS event-driven pipeline. Each morning, the system pulls your wellness stats from Garmin (steps/activity), Withings (weight), and MyFitnessPal (food), stores raw JSON in S3, transforms everything into analytics-ready Parquet via Glue, refreshes your Quicksight dashboards for easy trend tracking, and delivers a daily summary to your inbox via SNS.

All triggers, batching, and orchestration use best-practice AWS serverless event-driven constructs (CloudTrail, EventBridge, Glue). Hassle-free, scalable, and cost-efficient!

***

## 🛠 AWS Services Used

- **API Gateway** – Public endpoints for secure Lambda triggers to fetch API data
- **Lambda** – Custom Python pulls (Garmin, Withings, MyFitnessPal) to S3 (JSON)
- **S3** – Data lake for raw and processed daily metrics
- **CloudTrail** – Monitors S3 uploads for workflow events
- **EventBridge** – Batches events, triggers Glue when the set is ready
- **Glue Workflow** – ETL jobs + crawler deliver analytics-ready data in Parquet
- **Athena** – Query your metrics with SQL
- **Quicksight** – Interactive dashboard for activity, weight, food logs
- **SNS** – Email notifications summarizing yesterday’s health, delivered daily

***

## 🔄 Architecture Components

**Ingestion:**  
Lambda functions fetch your latest stats using API Gateway (OAuth credentials from AWS Secrets Manager), then store as:
- `health-metrics-raw/YYYY-MM-DD/garmin.json`
- `health-metrics-raw/YYYY-MM-DD/withings.json`
- `health-metrics-raw/YYYY-MM-DD/myfitnesspal.json`

**Event Triggering and Orchestration:**
- S3 raw ingest triggers CloudTrail PutObject events
- EventBridge batches, fires Glue workflow when all sources have arrived
- Glue ETL jobs join/transform data into Parquet
- Glue Crawler updates partition tables in Glue Data Catalog
- After ETL, Glue triggers an SNS topic: you get an email summary!

**Analytics & Visualization:**  
- Athena queries for health summaries, Quicksight dashboards for clear trends

***

## 📝 Example API Integration (Python)

Garmin:  
```python
import requests
GARMIN_API_URL = "https://api.garmin.com/wellness-api/rest/activity"
headers = {"Authorization": "Bearer "}
response = requests.get(GARMIN_API_URL, headers=headers)
with open("/tmp/garmin.json", "w") as f:
    f.write(response.text)
# upload to S3 with boto3
```

Withings:  
```python
import requests
WITHINGS_API_URL = "https://wbsapi.withings.net/measure"
params = {"action": "getmeas", "access_token": ""}
response = requests.get(WITHINGS_API_URL, params=params)
with open("/tmp/withings.json", "w") as f:
    f.write(response.text)
# upload to S3 with boto3
```

MyFitnessPal:  
```python
import requests
MFP_API_URL = "https://api.myfitnesspal.com/v2/diary"
headers = {"Authorization": "Bearer "}
response = requests.get(MFP_API_URL, headers=headers)
with open("/tmp/myfitnesspal.json", "w") as f:
    f.write(response.text)
# upload to S3 with boto3
```

***

## 🔍 Features

- Automated collection of steps/activity, weight, and calories 🙌
- Daily summary email of your health stats in your inbox 📧
- Analytics dashboards with Quicksight for trends and insights 📈
- Fully serverless, efficient, easy to maintain, and scalable 🚀

***

## 🌟 Future Enhancements

- Add more health APIs: Fitbit, Apple Health, Strava, you name it!
- Push notifications or Slack alerts for goals or anomalies
- Machine learning: automatically analyze trends and flag issues
- Family/multi-user support

***

## 🔒 Security Measures

- All API endpoints gated by API Gateway authentication
- Minimal IAM permissions throughout
- Data encryption in S3, secrets in Secrets Manager

***

## 📊 Performance Metrics

- End-to-end ingestion and transformation in under 10 minutes/day 🚦
- Dashboard refresh by 7:15am
- Email delivered by 8:00am

***

## 💻 Technical Requirements

- AWS account (with Glue, Quicksight, Athena enabled)
- Credentials for Garmin, Withings, MyFitnessPal
- Email subscribed to your SNS topic

***

## ⚠️ Limitations

- API rate limits & token expiry can affect freshness
- S3/Glue/Quicksight quotas apply
- Limited real-time capabilities (optimized for daily batch)

***

*Built with AWS best practices for cloud automation and analytics.*  
_Open to contributions & suggestions!_

***

### 📚 Reference

Uses workflow pattern from:  
[Build a serverless event-driven workflow with AWS Glue and Amazon EventBridge](https://aws.amazon.com/blogs/big-data/build-a-serverless-event-driven-workflow-with-aws-glue-and-amazon-eventbridge/)
