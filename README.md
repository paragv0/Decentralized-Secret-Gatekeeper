# Decentralized Secret Gatekeeper

**Author:** Parag Jagjivan Vadher  
**Context:** MSc Cloud Computing, National College of Ireland  

A serverless AWS security architecture that acts as a cryptographic proxy for sensitive cloud secrets. This project prevents unauthorized direct-database access and ensures **non-repudiation** by forcing all key retrievals through an immutable, hash-linked Audit Ledger (DLT).

---

## Links & Live Demos

* **Frontend Dashboard:** [Secure Gatekeeper UI (S3 Hosted)](https://x24222160-sc-api.s3.us-east-1.amazonaws.com/index.html)
* **Interactive API Docs:** [Swagger UI / FastAPI Docs](https://xfno7odld6hxji7kchsbpqo7za0jqtix.lambda-url.us-east-1.on.aws/docs)
* **API Gateway Base URL:** `https://lv1jxm0j6a.execute-api.us-east-1.amazonaws.com/Test/ledger`

---

## Overview

In standard cloud environments, developers often fetch API keys directly from a vault or, worse, hardcode them. This creates a massive security blind spot: if a key is leaked, there is no mathematical proof of who accessed it or when.

**The Gatekeeper** solves this by merging **AWS Secrets Manager** with a **DynamoDB-backed Distributed Ledger**. 
To get a secret, a user must authenticate via API Gateway and "mine" a block on the ledger. The system will *only* unlock the vault and return the secret if the cryptographic audit log is successfully and permanently recorded.

---

## Technologies Used

* **Cloud Provider:** Amazon Web Services (AWS)
* **Compute Layer:** AWS Lambda (Serverless Python 3.x)
* **API Routing:** Amazon API Gateway & AWS Lambda Function URLs
* **Database (The Ledger):** Amazon DynamoDB (NoSQL)
* **Security (The Vault):** AWS Secrets Manager
* **Languages & Frameworks:** Python, FastAPI, JavaScript, HTML5, CSS3
* **Libraries:** `boto3` (AWS SDK), `hashlib` (SHA-256 Cryptography)

---

## API Endpoints

The system exposes a RESTful interface protected by an AWS API Gateway Usage Plan. All requests require a valid `x-api-key` in the headers.

### `POST /ledger` (Authenticate & Fetch)
* **Purpose:** Mines a new block on the ledger and dispenses the requested secrets.
* **Payload:** Requires `project_id`, `user_id`, and `action`.
* **Behavior:** Fails immediately if the database write is unsuccessful, ensuring secrets are never dispensed without an audit trail.

### `GET /ledger` (Audit & Verify)
* **Purpose:** Retrieves the ledger history and mathematically verifies the cryptographic integrity of the chain.
* **Query Params:** `?project_id={Your_Project}`
* **Behavior:** Recalculates all SHA-256 hashes from the Genesis block to the current height. Returns a status of `VALID` or `TAMPERED` if any data anomalies are detected.

---

## For Colleagues: How to Use the API

If you have been issued an API Key for this project, you do not need to access the AWS Console. You can fetch your project's credentials dynamically at runtime.

### 1. Requirements
* Your unique **API Key** (Ask the repository owner).
* Your **Project ID** (e.g., `Project-Alpha`).

### 2. Python Implementation Example
Do not hardcode your keys. Inject them into your environment at runtime using this script:

```python
import requests
import os

# 1. Configuration
API_URL = "[https://lv1jxm0j6a.execute-api.us-east-1.amazonaws.com/Test/ledger](https://lv1jxm0j6a.execute-api.us-east-1.amazonaws.com/Test/ledger)"
API_KEY = "your_issued_api_key_here"
PROJECT_ID = "Project-Alpha"
USER_ID = "student_name"

# 2. Request the Vault
response = requests.post(
    API_URL, 
    headers={
        "Content-Type": "application/json",
        "x-api-key": API_KEY
    },
    json={
        "project_id": PROJECT_ID,
        "user_id": USER_ID,
        "action": "Initializing backend services"
    }
)

data = response.json()

# 3. Handle the Response
if response.status_code == 200:
    print(f"Success! Block mined with Hash: {data['ledger_hash']}")
    
    # Parse the returned keychain
    import json
    secrets = json.loads(data['API_KEY'])
    
    # Inject into environment (safest method)
    os.environ['OPENAI_API_KEY'] = secrets.get('OPENAI_API_KEY')
    os.environ['DB_PASSWORD'] = secrets.get('DB_PASSWORD')
else:
    print(f"Access Denied: {data}")
---

## The Verification Dashboard

This project includes a local, zero-dependency `index.html` dashboard. You can run this file directly in any web browser to:
1. **Fetch Secrets visually:** Generate a block and retrieve your keychain without writing code.
2. **Audit the Ledger:** View the real-time, immutable history of vault access.
3. **Cryptographic Verification:** Run a simulated terminal sequence to recalculate the SHA-256 hashes of the entire DynamoDB table to mathematically prove the database has not been tampered with.

*Note: You must click the "Settings" (Gear) icon in the top right to input your API Key before the dashboard will connect to AWS.*

---

## Security Model & Principles

* **Ephemeral Memory:** Secrets are returned as temporary JSON strings and exist only in the RAM of the requesting application. They never touch the disk.
* **IAM Least Privilege:** The Lambda function has a strict inline policy limiting its `secretsmanager:GetSecretValue` permission to specific ARNs. It cannot read the entire vault.
* **Tamper-Evident:** Every log entry includes a `PreviousHash`. If a rogue administrator alters a DynamoDB item manually, the subsequent block's hash will break, and the GET Verification endpoint will flag the chain as `TAMPERED`.
* **The "Valet Key" Pattern:** Users authenticate with a revocable API Gateway key. If a user's API key is leaked, it can be revoked instantly without requiring a rotation of the actual Master Secrets stored in the vault.
