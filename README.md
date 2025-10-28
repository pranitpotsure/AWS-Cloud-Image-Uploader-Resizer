# ☁️ AWS Cloud Image Uploader & Resizer

> **A fully serverless cloud project built with AWS Lambda, S3, and API Gateway — featuring automatic image resizing and a modern static upload portal.**

---

## 🏗️ Architecture Overview

[User Browser]
↓
[S3 Static Website (Frontend)]
↓ (POST)
[API Gateway]
↓
[Lambda Upload Function]
↓
[S3 Source Bucket]
↓ (Trigger)
[Lambda Resizer Function]
↓
[S3 Resized Output Bucket]


---

## 🧠 About the Project

| Feature | Description |
|----------|--------------|
| 🌐 **Frontend** | Static web app hosted on Amazon S3 |
| ⚙️ **Backend** | Serverless APIs via AWS Lambda + API Gateway |
| 🖼️ **Automation** | Image resizing via S3 event trigger |
| ☁️ **Storage** | Two S3 buckets – source and resized |
| 🔒 **Security** | IAM roles with least-privilege access |
| 🧩 **Architecture Type** | Fully serverless, scalable, and event-driven |

---

## 🪄 Tech Stack & Tools

![AWS](https://img.shields.io/badge/AWS-FF9900?logo=amazon-aws&logoColor=white)
![Lambda](https://img.shields.io/badge/AWS%20Lambda-FF9900?logo=awslambda&logoColor=white)
![S3](https://img.shields.io/badge/Amazon%20S3-569A31?logo=amazons3&logoColor=white)
![API Gateway](https://img.shields.io/badge/API%20Gateway-EC7211?logo=amazonapigateway&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?logo=javascript&logoColor=black)
![CloudWatch](https://img.shields.io/badge/CloudWatch-FF4F8B?logo=amazonaws&logoColor=white)

---

## 📂 Folder Structure
aws-image-uploader/
│
├── frontend/
│ ├── index.html
│ ├── style.css
│ ├── script.js
│ └── assets/
│ ├── aws-s3.png
│ ├── lambda.png
│ ├── api-gateway.png
│
├── backend/
│ ├── lambda_upload.py
│ └── lambda_resizer.py
│
└── README.md


---

## 🔧 Setup Guide (From Scratch)

### 🪣 1. Create Two S3 Buckets
| Bucket Name | Purpose |
|--------------|----------|
| `image-upload-source` | Stores uploaded original images |
| `image-resized-output` | Stores resized images |

➡️ Enable **static website hosting** for your frontend bucket.

---

### 🔐 2. Create IAM Role for Lambda

Attach the following **least-privilege policy**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3Access",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": [
        "arn:aws:s3:::image-upload-source/*",
        "arn:aws:s3:::image-resized-output/*"
      ]
    }
  ]
}
```
---

### ⚙️ 3. Lambda Function – Upload to S3

Filename: lambda_upload.py
```
import json
import boto3
import base64

s3 = boto3.client('s3')
BUCKET_NAME = 'image-upload-source'

def lambda_handler(event, context):
    try:
        file_content = base64.b64decode(event['body'])
        file_name = event['headers']['filename']
        
        s3.put_object(Bucket=BUCKET_NAME, Key=file_name, Body=file_content)

        return {
            'statusCode': 200,
            'headers': {'Content-Type': 'application/json'},
            'body': json.dumps({
                'message': '✅ Upload successful!',
                'file_url': f"https://{BUCKET_NAME}.s3.amazonaws.com/{file_name}"
            })
        }
    except Exception as e:
        return {'statusCode': 500, 'body': json.dumps({'error': str(e)})}
```
---

### 🌐 4. API Gateway Setup

Create a REST API
Add a new resource: /upload
Add method: POST → Integration Type: Lambda Function
Enable CORS
Deploy the API and note your endpoint URL
(e.g. https://xyz123.execute-api.ap-south-1.amazonaws.com/prod/upload)

---

### 🖼️ 5. Lambda Function – Image Resizer

Filename: lambda_resizer.py
```
import boto3
from PIL import Image
import io

s3 = boto3.client('s3')

def lambda_handler(event, context):
    source_bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    destination_bucket = 'image-resized-output'

    img_obj = s3.get_object(Bucket=source_bucket, Key=key)
    image = Image.open(img_obj['Body'])
    image = image.resize((300, 300))

    buffer = io.BytesIO()
    image.save(buffer, 'JPEG')
    buffer.seek(0)

    s3.put_object(
        Bucket=destination_bucket,
        Key=f"resized-{key}",
        Body=buffer,
        ContentType='image/jpeg'
    )
```
---

#### 🪄 Trigger:
  Go to S3 → image-upload-source → Properties → Event notifications → Add trigger
    → Event type: All object create events
    → Lambda function: lambda_resizer
--- 

### 💻 6. Frontend Setup
Edit your script.js:
const apiUrl = "https://your-api-id.execute-api.ap-south-1.amazonaws.com/prod/upload";
Host the frontend
Upload index.html, CSS, JS, and icons to your S3 static website bucket.
---

### 🌈 User Flow
Open the static upload portal (S3 website URL)
Select or drag-drop an image
Click Upload — image goes via API Gateway → Lambda → uploaded to S3
The S3 event triggers Lambda Resizer
Resized image is stored in image-resized-output
You can view both original and resized images via S3 URLs
---

### 🧩 Architecture Highlights
Layer	Service	Function
Frontend	S3 Static Website	File upload interface
API	API Gateway	Routes requests
Compute	AWS Lambda	Upload + Resize logic
Storage	S3 Buckets	Store images
Monitoring	CloudWatch	Logs + metrics
Security	IAM	Role-based access
---

### 💡 Key Learnings
Designed and deployed a serverless image pipeline
Built REST APIs using Lambda + API Gateway
Implemented event-driven automation
Learned IAM security and S3 permissions
Designed a modern AWS-branded web interface
---

### 🧠 Future Enhancements
Add CloudFront CDN for faster delivery
Add image format conversion (PNG/JPEG) options
Add progress bar + image preview before upload
Store metadata in DynamoDB
---

### ✨ Author
👨‍💻 Pranit Potsure
AWS • Cloud • DevOps Enthusiast
📫 GitHub
 | 🌐 AWS Cloud Portfolio 🚀
---

### 📸 Preview
