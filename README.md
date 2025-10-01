
# 🚀 Building an End-to-End AWS Application with Amplify, Lambda, API Gateway & DynamoDB

Building this hands-on project will enhance your technical know-how on how to combine some of the AWS services such as **Amplify**, **Lambda Function**, **IAM**, **DynamoDB**, and the **API Gateway** services to build and deploy an end-to-end application.

---

## Step 1: Create and Host a Web Page

First question: **How do we even host this application???**

👉 The answer is simple: **AWS Amplify!**

In simple terms, **AWS Amplify** is a comprehensive tool that helps developers build and deploy scalable mobile apps and web applications quickly and easily.

* Open the **Amplify service** in the AWS Console.
* Connect GitHub (or upload a zip folder if you prefer).
* If using GitHub, a **third-party authentication** will be needed. Grant approval, then choose your repository.
* Deploy the project.

🎉 **Congratulations!** Your site has been deployed successfully!

We now have a **live webpage**. Next, let’s work on doing some math using **Lambda**.

---

## Step 2: Do Some Math Using AWS Lambda

**AWS Lambda** is a compute service that runs code without managing servers. Your code runs automatically and scales up/down as needed.

Steps:

1. In the console, open **Lambda Service**.
2. Create a new function.
3. Choose **Python** as the runtime.
4. Replace the default code with the following:

```python
import json
import math

def lambda_handler(event, context):
    mathResult = math.pow(int(event['base']), int(event['exponent']))
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': f"Your result is {mathResult}"
        })
    }
```

* Save & deploy.
* Test with sample inputs.
* ✅ Expected output: `statusCode: 200` and the correct math result.

---

## Step 3: Trigger Lambda Using API Gateway

**Amazon API Gateway** allows you to create, publish, maintain, and secure REST, HTTP, and WebSocket APIs at scale. Perfect for invoking Lambda functions.

Steps:

1. Open **API Gateway** service.
2. Create a **REST API**.
3. Create a **POST Method** → link it to your Lambda function.
4. Enable **CORS**.
5. Deploy API → create a new stage.
6. Copy the **Invoke URL**.

✅ You can now test your API by sending POST requests with inputs.

---

## Step 4: Store Results in DynamoDB

**Amazon DynamoDB** is a fully managed NoSQL database with millisecond performance.

Steps:

1. Open **DynamoDB Service**.
2. Create a table (with `ID` as primary key).
3. Copy the **ARN**.
4. Update your **IAM Role** for Lambda → Add an inline policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor0",
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:DeleteItem",
        "dynamodb:GetItem",
        "dynamodb:Scan",
        "dynamodb:Query",
        "dynamodb:UpdateItem"
      ],
      "Resource": "YOUR-TABLE-ARN"
    }
  ]
}
```

---

### Update Lambda to Write to DynamoDB

```python
import json
import math
import boto3
from time import gmtime, strftime

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('PowerOfMathDatabase')
now = strftime("%a, %d %b %Y %H:%M:%S +0000", gmtime())

def lambda_handler(event, context):
    mathResult = math.pow(int(event['base']), int(event['exponent']))
    response = table.put_item(
        Item={
            'ID': str(mathResult),
            'LatestGreetingTime': now
        })
    return {
        'statusCode': 200,
        'body': json.dumps('Your result is ' + str(mathResult))
    }
```

* Replace **table name** with your actual DynamoDB table.
* Save & deploy.
* Test again → check DynamoDB to confirm records are saved.

---

## Step 5: Connect Amplify Frontend to API Gateway

Create a simple **frontend UI** and connect it to your API Gateway URL:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>To the Power of Math!</title>
    <style>
        body { background-color: #222629; }
        h1 { color: #FFF; margin-left: 20px; }
        label { color: #86C232; font-size: 20px; margin-left: 20px; }
        input { font-size: 20px; margin: 10px; width: 100px; }
        button {
            background-color: #86C232; color: #FFF; font-weight: bold;
            margin-left: 30px; width: 140px;
        }
    </style>
    <script>
        var callAPI = (base, exponent) => {
            var myHeaders = new Headers();
            myHeaders.append("Content-Type", "application/json");
            var raw = JSON.stringify({"base": base, "exponent": exponent});
            var requestOptions = {
                method: 'POST',
                headers: myHeaders,
                body: raw
            };
            fetch("https://YOUR-API-URL.amazonaws.com/dev", requestOptions)
            .then(response => response.text())
            .then(result => alert(JSON.parse(result).body))
            .catch(error => console.log('error', error));
        }
    </script>
</head>
<body>
    <h1>TO THE POWER OF MATH!</h1>
    <form>
        <label>Base number:</label>
        <input type="text" id="base">
        <label>...to the power of:</label>
        <input type="text" id="exponent">
        <button type="button"
            onclick="callAPI(document.getElementById('base').value, document.getElementById('exponent').value)">
            CALCULATE
        </button>
    </form>
</body>
</html>
```

* Replace `"https://YOUR-API-URL.amazonaws.com/dev"` with your API Gateway Invoke URL.
* Push changes to GitHub.
* Amplify auto-redeploys (thanks to **CI/CD**).

---

## 🎯 Final Notes

* Test the app → enter numbers → check DynamoDB for results.
* Don’t forget to **delete AWS resources** after testing to avoid costs.

