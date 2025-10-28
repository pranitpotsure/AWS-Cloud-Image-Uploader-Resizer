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
