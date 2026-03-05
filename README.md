# Ledgr
Lightweight serverless expense tracker.
Requires: AWS + Grafana Cloud (Free Tier)

## Why I Built This
I needed a quick and easy way to track my monthly spending and answer questions like:
- *"What have I spent in total this month?"*
- *"What am I spending my money on?"*
- *"How many times per month am I going out to purchase things?"*
- *"What am I spending the most money on?"*
- *"How many times a month am I spending money at bars or cafes?"*
- *"What are my spending trends?"*
- **"Where can I cut back on spending?"**

The goal was to build a dashboard to view monthly spending, category breakdowns, insights into top vendors, and trends.

**..Picture Coming Soon..**


## Architecture
**Logging Flow**
iOS Shortcut --> API Gateway --> DynamoDB --> Streams --> Lambda(Categorize & Sum) --> DynamoDB

**Visualization Flow**
Grafana --> Athena --> Lambda (DB Connector) --> DynamoDB

#### DynamoDB Structure
id (n): YYYYMM (Primary Key)
timestamp (n): $context.requestTimeEpoch (Sort Key)
---
category (s): Category of expense. When pulled via Lambda, please python.string.upper()
description (s): Likely the name of the vendor
amount (n): Value of expense


#### Athena Setup
**Athena Query on DynamoDB via Athena Federated Query and DynamoDB Connector**
1. Deploy DynamoDB Connector
```aws serverlessrepo create-cloud-formation-change-set \
  --application-id arn:aws:serverlessrepo:us-east-1:292517598671:applications/AthenaDynamoDBConnector \
  --stack-name athenaDynamoDBConnector \
  --capabilities CAPABILITY_RESOURCE_POLICY CAPABILITY_IAM \
  --parameter-overrides '[{"Name":"AthenaCatalogName","Value":"dynamo"},{"Name":"SpillBucket","Value":"karl-macro-tracker-athena-spill-bucket"}]'
```

2. Register Connector as Data Catalog in Athena
```ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
REGION=$(aws configure get region)

aws athena create-data-catalog \
  --name dynamo \
  --type LAMBDA \
  --description "DynamoDB Federated Connector" \
  --parameters function=arn:aws:lambda:${REGION}:${ACCOUNT_ID}:function:dynamo
```

3. Create Dashboard & Panels in Grafana
How it works: Grafana --> Athena --> S3

*For large tables queried frequently, it's often cheaper to export DynamoDB data to S3 (via DynamoDB Streams or point-in-time exports) and query from there with Athena at standard S3 rates* --> **Evaluate later to determine if this is necessary. Check Cost Explorer**


---
# Todo
1. DONE! Month-over-month change: % change from last month
2. NEED MORE DATA! Rolling 3 month average
3. Line chart for category trends over time
4. Top vendors per category
5. Update queries to use Grafana's Time Range feature (macros)

