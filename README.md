## Serverless AWS Guestbook

A simple serverless guestbook application built entirely on AWS managed services. Visitors can leave their name and a message, and see previous entries displayed on the page. No servers to manage — everything scales automatically and costs close to nothing at low traffic.

## Live Demo

http://montyrawal7-guestbook.s3-website-us-east-1.amazonaws.com

## Architecture

The frontend is a static HTML, CSS, and JavaScript page hosted on Amazon S3 with static website hosting enabled. It calls an API built with Amazon API Gateway (HTTP API), which has two routes:

POST /submit → triggers a Lambda function that validates the input and writes a new entry to DynamoDB
GET /messages → triggers a second Lambda function that scans the DynamoDB table and returns all entries as JSON

The frontend fetches /messages on page load to render existing entries, and calls /submit when the form is submitted, then refreshes the list.

![Architecture Diagram](https://github.com/montyrawal7/serverless_cloud_project_1/blob/main/Project_Diagram.png?raw=true)

## Tech stack

Amazon S3 for static website hosting. 

Amazon API Gateway for the REST API. 

AWS Lambda, running Node.js or Python, for the backend logic. 

Amazon DynamoDB as the database, chosen because it requires no schema and scales automatically. 

AWS IAM for permissions between these services.

## Repository structure
```
serverless_cloud_project_1/
├── README.md
├── Project_Diagram.png
├── frontend/
│   ├── index.html
│   ├── styles.css
│   └── script.js
└── backend/
    ├── submit_handler.py
    ├── get_messages_handler.py
    ├── submit_handler_policy.json
    ├── get_messages_handler_policy.json
    └── bucket_policy.json
```
## Setup and deployment

DynamoDB — Created a table named guestbook with partition key id (String), on-demand capacity mode.

Lambda — Created two Python 3.12 functions: submitHandler (using submit_handler.py) and getMessagesHandler (using get_messages_handler.py). Added an environment variable TABLE_NAME=guestbook to both.

IAM — Attached an inline policy to submitHandler's execution role granting dynamodb:PutItem scoped to the table's ARN. Attached a separate policy to getMessagesHandler's role granting dynamodb:Scan, also scoped to the table's ARN. See the policy JSON files in /backend.

API Gateway — Created an HTTP API. Added a POST /submit route integrated with submitHandler, and a GET /messages route integrated with getMessagesHandler. Enabled CORS: allow origins *, methods GET, POST, OPTIONS, headers Content-Type.

Frontend — In script.js, set API_BASE_URL to your API Gateway invoke URL.

S3 — Created a bucket, enabled static website hosting (index document: index.html), attached a public-read bucket policy (see bucket_policy.json), and uploaded the three frontend files.

## Security Notes

Each Lambda's IAM role is scoped to only the single DynamoDB action it needs (PutItem or Scan) on only the guestbook table's ARN — no broader DynamoDB or account access.
All user-submitted text (name, message) is escaped client-side before being rendered, to prevent stored XSS from malicious guestbook entries.
Input is trimmed and length-capped server-side in both Lambda functions.

## Cost

Built entirely on pay-per-use services (S3, Lambda, API Gateway, DynamoDB on-demand), so cost scales with traffic and is effectively $0 at rest. All four services fall within AWS free-tier limits for a low-traffic demo project.
