## Serverless AWS Guestbook

A simple serverless guestbook application built entirely on AWS managed services. Visitors can leave their name and a message, and see previous entries displayed on the page. No servers to manage — everything scales automatically and costs close to nothing at low traffic.

## Architecture

The frontend is a static HTML, CSS, and JavaScript page hosted on Amazon S3 with static website hosting enabled. The frontend calls an API built with Amazon API Gateway, which has two routes. A POST route to slash submit accepts new guestbook entries and triggers an AWS Lambda function that writes the name and message into an Amazon DynamoDB table. A GET route to slash messages triggers a second Lambda function that scans the DynamoDB table and returns all entries as JSON, which the frontend then renders on the page.

## Tech stack

Amazon S3 for static website hosting. Amazon API Gateway for the REST API. AWS Lambda, running Node.js or Python, for the backend logic. Amazon DynamoDB as the database, chosen because it requires no schema and scales automatically. AWS IAM for permissions between these services.

## Repository structure

At the root there's an index HTML file, a styles CSS file, and a script JS file for the frontend. There's also a backend folder containing two Lambda function files, one named submit handler and one named get messages handler.

## Setup and deployment

First, create the DynamoDB table. Name it guestbook and set the partition key to id, of type string.

Second, create the two Lambda functions. Create one function named submit handler using the code from the backend folder, and give it an environment variable called TABLE underscore NAME set to guestbook. Then create a second function named get messages handler the same way, with the same environment variable.

Third, attach IAM permissions. Each Lambda needs an execution role. The submit handler needs DynamoDB put item permission on the guestbook table. The get messages handler needs DynamoDB scan permission on the same table. Keep these permissions scoped only to that one table's resource, not to all of DynamoDB.

Fourth, create the API Gateway. Set up a REST or HTTP API with a POST route at slash submit connected to the submit handler Lambda, and a GET route at slash messages connected to the get messages handler Lambda. Enable CORS on both routes since the frontend will call this API from a different origin.

Fifth, update the frontend. In script JS, set the API base URL to your API Gateway invoke URL. On page load, the script should call the messages endpoint and render the results. On form submission, it should call the submit endpoint and then refresh the list.

Sixth, deploy the frontend. Create an S3 bucket, enable static website hosting, upload the three frontend files, and attach a bucket policy allowing public read access to the objects.

## Testing

You can test the backend directly without the frontend using curl. A post request with a JSON body containing name and message fields sent to your slash submit endpoint should return a success response, and a get request to slash messages should return the entries you've submitted as JSON.

## Security notes

Sanitize and escape any user-submitted name or message text before rendering it on the page, to avoid stored cross site scripting. Keep IAM permissions scoped to only the specific actions and table each function needs.

## Cleanup

To avoid ongoing charges, delete the API Gateway API, both Lambda functions, the DynamoDB table, and empty and delete the S3 bucket when you're done with the project.

## License

MIT License.
