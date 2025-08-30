# Simple Serverless Feedback Application (AWS)

A fully serverless feedback app powered by **S3 (static site)**, **API Gateway**, **Lambda**, and **DynamoDB**. The frontend posts JSON to an API endpoint; Lambda writes the item to DynamoDB.

## Architecture
- S3 static website hosting serves `index.html`
- API Gateway (HTTP API) exposes the `/feedback` endpoint
- Lambda function receives the POST and writes to DynamoDB
- IAM roles grant least‑privilege access for Lambda and DynamoDB

> Add your diagram image to `screenshots/architecture.png` and embed it below.

![Architecture](screenshots/architecture.png)

## Implementation (Condensed)
1. **IAM**
   - Create an execution role for Lambda with basic logging.
   - Grant DynamoDB permissions to put items into your table.

2. **S3 Bucket (Static Website)**
   - Enable static website hosting.
   - Upload `index.html` (see below).
   - Optional public read bucket policy (adjust bucket name):
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Sid": "PublicReadGetObject",
           "Effect": "Allow",
           "Principal": "*",
           "Action": "s3:GetObject",
           "Resource": "arn:aws:s3:::YOUR_BUCKET/*"
         }
       ]
     }
     ```

3. **Frontend Form (`index.html`)**
   Update `apiUrl` with your API Gateway endpoint:
   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
     <meta charset="UTF-8">
     <title>Feedback Form</title>
   </head>
   <body>
     <h2>Submit Feedback</h2>
     <form id="feedbackForm">
       <input type="text" id="name" placeholder="Your Name" required><br><br>
       <textarea id="message" placeholder="Your Message" required></textarea><br><br>
       <button type="submit">Submit</button>
     </form>

     <p id="response"></p>

     <script>
       const apiUrl = "YOUR_API_URL"; // e.g., https://abc123.execute-api.us-east-1.amazonaws.com/feedback
       document.getElementById("feedbackForm").addEventListener("submit", async (e) => {
         e.preventDefault();
         const name = document.getElementById("name").value;
         const message = document.getElementById("message").value;

         const response = await fetch(apiUrl, {
           method: "POST",
           headers: {"Content-Type": "application/json"},
           body: JSON.stringify({ name, message })
         });

         const result = await response.json();
         document.getElementById("response").innerText = result.message || "Submitted!";
       });
     </script>
   </body>
   </html>
   ```

4. **DynamoDB**
   - Create a table (e.g., `feedback`) with a partition key like `id` or `name`.
   - Optionally add a sort key or TTL for cleanup.

5. **Lambda**
   - Runtime: Python/NodeJS.
   - Minimal handler pseudocode:
     ```python
     import json, os, boto3, uuid, datetime
     dynamo = boto3.resource("dynamodb").Table(os.environ["TABLE_NAME"])

     def handler(event, context):
         body = json.loads(event.get("body") or "{}")
         item = {
             "id": str(uuid.uuid4()),
             "name": body.get("name"),
             "message": body.get("message"),
             "createdAt": datetime.datetime.utcnow().isoformat()
         }
         dynamo.put_item(Item=item)
         return {"statusCode": 200, "headers": {"Content-Type": "application/json"}, "body": json.dumps({"message": "Thanks for the feedback!"})}
     ```
   - Configure environment variable `TABLE_NAME` and grant write permissions.

6. **API Gateway**
   - Create an HTTP API.
   - Integrate it with Lambda and route `POST /feedback` to your function.

## Validate / Output
- Open the S3 website URL, submit the form, and verify the item appears in the DynamoDB table.
- Export items to CSV from the console for quick verification.

## Repo Structure
```
serverless-feedback-app/
├─ README.md
├─ index.html
├─ policy.json
└─ screenshots/
   └─ architecture.png   # (add your diagram)
```

## Future Enhancements
- Input validation and form UX improvements.
- Add CORS config on API Gateway for broader origins.
- API keys/usage plans or Cognito for auth.
- CI/CD for Lambda with SAM/Serverless Framework.
