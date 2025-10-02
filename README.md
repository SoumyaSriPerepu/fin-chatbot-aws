# financial-chatbot-aws
financial ai agent in a chatbot in was


Cloud-native **Financial Q&A + Forecasting Chatbot** on AWS.  
No installs on your Mac — deploy via **GitHub Actions (OIDC)**, use **SageMaker Studio** (browser) for the model endpoint, and host UI on **Amplify**.

> Educational demo only. Not investment advice.

---

## What you get

- Daily OHLCV + news **ingest** → S3 (Athena-ready)
- RAG-lite **news retriever**
- **ARIMA** forecast on **SageMaker** endpoint
- Single **/ask** API (API Gateway + Lambda + DynamoDB cache)
- Minimal **Next.js** UI (Amplify Hosting)

---

## 0) Create the GitHub repo

1. Make a repo `fin-chatbot-aws` on GitHub.
2. Upload this entire folder structure (use the web UI or Codespaces).

---

## 1) Allow GitHub Actions to deploy to AWS (OIDC)

### A) Add OIDC provider (AWS Console)
- IAM → **Identity providers** → **Add provider**
  - Provider type: **OpenID Connect**
  - Provider URL: `https://token.actions.githubusercontent.com`
  - Audience: `sts.amazonaws.com`

### B) Create IAM Role for Actions
- IAM → **Roles** → **Create role**
  - Trusted entity: **Web identity**
  - Identity provider: the one you added
  - Audience: `sts.amazonaws.com`
  - **Condition (repo limit)**:  
    Key: `token.actions.githubusercontent.com:sub`  
    Condition: `StringLike`  
    Value: `repo:<YOUR_GH_USERNAME>/fin-chatbot-aws:*`
- **Permissions** (quick start; tighten later):
  - `AWSCloudFormationFullAccess`
  - `AmazonS3FullAccess`
  - `AWSLambda_FullAccess`
  - `AmazonAPIGatewayAdministrator`
  - `AmazonDynamoDBFullAccess`
  - `CloudWatchFullAccess`
  - `IAMReadOnlyAccess`
  - `AmazonSageMakerFullAccess`
- Name: **GitHubActionsFinchatRole** → Create.
- Copy the **Role ARN**.

### C) Push the workflow (already in `.github/workflows/deploy-backend.yml`)
- Edit the file to paste your **Role ARN** and region (default `us-east-1`).
- Commit to `main` → Action runs → Stack deploys.

**Outputs to note (from Actions logs / CloudFormation):**
- `ApiUrl` (HTTP POST `/ask`)
- `DataLakeBucketName`
- `CacheTableName`

---

## 2) Create SageMaker endpoint (browser only)

1. SageMaker → **Domains** → create Domain (Quick setup) if first time.
2. **Launch Studio** → **File > New > Terminal**.
3. Run:
   ```bash
   git clone https://github.com/<YOUR_GH_USERNAME>/fin-chatbot-aws.git
   cd fin-chatbot-aws
   python sagemaker/deploy_sagemaker.py
