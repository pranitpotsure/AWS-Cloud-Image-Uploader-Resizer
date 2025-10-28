# ☁️ AWS Cloud Image Uploader & Resizer

> **A fully serverless cloud project built with AWS Lambda, S3, and API Gateway — featuring automatic image resizing and a modern static upload portal.**

---

## 🏗️ Architecture Overview
```
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
```

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
```
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
```

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

Filename: generate-presign.py
```
import json
import boto3

s3 = boto3.client('s3')
BUCKET_NAME = "pranit-image-upload-source"

def lambda_handler(event, context):
    print("EVENT:", event)

    try:
        # Parse request body
        body = json.loads(event.get("body", "{}"))
        filename = body.get("filename")
        content_type = body.get("content_type")

        if not filename:
            return {"statusCode": 400, "body": json.dumps({"error": "filename required"})}

        # ✅ Generate presigned URL (PUT method)
        presigned_url = s3.generate_presigned_url(
            "put_object",
            Params={
                "Bucket": BUCKET_NAME,
                "Key": filename,
                "ContentType": content_type,  # ✅ include this
            },
            ExpiresIn=300  # 5 minutes
        )

        return {
            "statusCode": 200,
            "headers": {
                "Access-Control-Allow-Origin": "https://pranit-image-web.s3.ap-south-1.amazonaws.com",
                "Access-Control-Allow-Methods": "POST, OPTIONS",
                "Access-Control-Allow-Headers": "Content-Type"
            },
            "body": json.dumps({
                "upload_url": presigned_url,
                "key": filename
            })
        }

    except Exception as e:
        print("Error:", e)
        return {
            "statusCode": 500,
            "headers": {
                "Access-Control-Allow-Origin": "https://pranit-image-web.s3.ap-south-1.amazonaws.com"
            },
            "body": json.dumps({"error": str(e)})
        }

```
---

### 🌐 4. API Gateway Setup (HTTP API)
**Type:** HTTP API (lightweight, cost-effective)  
**Integration:** Lambda (`image-presign-lambda`)  
**Route:** `POST /presign`  
 
 ⚙️ CORS Settings
- **Allow origins:** `https://pranit-image-web.s3.ap-south-1.amazonaws.com`
- **Allow methods:** `OPTIONS, POST`
- **Allow headers:** `Content-Type`
- **Allow credentials:** ❌ Disabled

 🚀 Deploy
- **Stage:** `prod`
- **Invoke URL:**  
  `https://8rfuwftbnd.execute-api.ap-south-1.amazonaws.com`

🧪 Test Command
```bash
curl -X POST https://8rfuwftbnd.execute-api.ap-south-1.amazonaws.com/presign \
  -H "Content-Type: application/json" \
  -d '{"filename":"test.jpg","content_type":"image/jpeg"}'
```

---

### 🖼️ 5. Lambda Function – Image Resizer

Filename: lambda_resizer.py
```
import boto3
from PIL import Image
import io
import os

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # --- S3 event info ---
    source_bucket = event['Records'][0]['s3']['bucket']['name']
    source_key = event['Records'][0]['s3']['object']['key']

    # --- Target bucket (fixed name or env var) ---
    target_bucket = os.environ.get('TARGET_BUCKET', 'pranit-image-upload-output-20251027')

    # --- Download image from S3 ---
    response = s3.get_object(Bucket=source_bucket, Key=source_key)
    image = Image.open(response['Body'])

    # --- Resize to 300x300 ---
    image = image.resize((300, 300))

    # --- Convert RGBA to RGB if needed ---
    if image.mode == 'RGBA':
        image = image.convert('RGB')

    # --- Save to buffer as JPEG ---
    buffer = io.BytesIO()
    image.save(buffer, 'JPEG')
    buffer.seek(0)

    # --- Upload resized image ---
    output_key = f"resized/{os.path.splitext(source_key)[0]}.jpg"
    s3.put_object(Bucket=target_bucket, Key=output_key, Body=buffer, ContentType='image/jpeg')

    return {
        'statusCode': 200,
        'body': f"✅ Image resized and uploaded to {target_bucket}/{output_key}"
    }

```
---

### 🪄 Trigger:
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
1.Open the static upload portal (S3 website URL)
2.Select or drag-drop an image
3.Click Upload — image goes via API Gateway → Lambda → uploaded to S3
4.The S3 event triggers Lambda Resizer
5.Resized image is stored in image-resized-output
6.You can view both original and resized images via S3 URLs

---

### 🧩 Architecture Highlights
  - layer: Frontend
    service: S3 Static Website
    function: File upload interface

  - layer: API
    service: API Gateway
    function: Routes requests

  - layer: Compute
    service: AWS Lambda
    function: Upload + Resize logic

  - layer: Storage
    service: S3 Buckets
    function: Store images

  - layer: Monitoring
    service: CloudWatch
    function: Logs + metrics

  - layer: Security
    service: IAM
    function: Role-based access


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
