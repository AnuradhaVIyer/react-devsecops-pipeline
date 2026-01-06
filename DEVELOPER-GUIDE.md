# **PHASE 1: Create React App & Git locally**

### **1️⃣ Create React app**

```
npx create-react-app react-devsecops-pipeline  
cd react-devsecops-pipeline
```

Run once locally:

`npm start`

---

### **2️⃣ Initialize Git repository**

```
git init  
git status
```

---

### **3️⃣ First commit (baseline)**

```
git add .  
git commit -m "Initial commit: Create React app"
```

---

### **4️⃣ Create initial tag**

```
git tag v0.1.0
```

Tag meaning: *initial stable baseline*

---

# **PHASE 2: Feature change on main branch**

### **5️⃣ Modify files (example)**

Edit `src/App.js`
```
function App() {  
  return (  
    <div>  
      <h1>React CI/CD Demo</h1>  
      <p>Initial version</p>  
    </div>  
  );  
}
```

```
git add src/App.js  
git commit -m "Update homepage text"
```
---

### **6️⃣ Tag release**
```
git tag v0.2.0
```
---

# **PHASE 3: Bugfix branch workflow**

### **7️⃣ Create bugfix branch**
```
git branch bugfix/header-typo  
git checkout bugfix/header-typo
```
---

### **8️⃣ Make bugfix change**

Fix typo or styling in `App.js`
```
<h1>React CI/CD Demo App</h1>
```
```
git add src/App.js  
git commit -m "Fix header typo"
```
---

### **9️⃣ Merge bugfix into main**
```
git checkout main  
git merge bugfix/header-typo
```
---

### **🔟 Tag bugfix release**
```
git tag v0.2.1
```
✅ **Local Git best practice complete**

---

# **PHASE 4: Push to GitHub**

### **1️⃣ Create GitHub repo (empty)**

Example:

https://github.com/your-org/react-devsecops-pipeline

---

### **2️⃣ Add remote & push**
```
git remote add origin https://github.com/your-org/react-devsecops-pipeline.git  
git push -u origin main  
git push origin --tags
```
---

# **PHASE 5: GitHub Actions – CI pipeline**

Create folder:
```
mkdir -p .github/workflows
```
---

## **CI Workflow (build, test, static + security scan)**

📄 `.github/workflows/ci.yml`
```
name: CI Pipeline

on:  
  pull_request:  
    branches: [ main ]  
  push:  
    branches: [ main ]

jobs:  
  build-test:  
    runs-on: ubuntu-latest

    steps:  
      - name: Checkout code  
        uses: actions/checkout@v4

      - name: Setup Node  
        uses: actions/setup-node@v4  
        with:  
          node-version: 18

      - name: Install dependencies  
        run: npm ci

      - name: Run tests  
        run: npm test -- --watchAll=false

      - name: Build app  
        run: npm run build

      # Static Analysis  
      - name: ESLint  
        run: npm run lint || true

      # Security – Dependency Scan  
      - name: NPM Audit  
        run: npm audit --audit-level=high || true
```
📌 **What this covers**

* Build validation  
* Unit tests  
* Static code analysis  
* Dependency vulnerability scan

---

# **PHASE 6: Auto-label Pull Requests**

📄 `.github/labeler.yml`
```
bugfix:  
  - "bugfix/**"

ci:  
  - ".github/workflows/**"

frontend:  
  - "src/**"
```
📄 `.github/workflows/labeler.yml`
```
name: PR Auto Labeler

on:  
  pull_request:

jobs:  
  label:  
    runs-on: ubuntu-latest  
    steps:  
      - uses: actions/labeler@v5
```
---

# **PHASE 7: CR (Change Request) branch flow**

### **Create CR branch**
```
git checkout -b cr/update-footer
```
Modify `App.js`:
```
<footer>© 2026 React CI/CD Demo</footer>
```
```
git add .  
git commit -m "CR: Add footer text"  
git push origin cr/update-footer
```
---

### **Create Pull Request**

* CI automatically runs  
* Labels auto-applied  
* Code review happens  
* Merge to `main`

✅ **CI runs again on merge**

---

# **PHASE 8: CD – Build & Deploy to AWS S3**

## **S3 Setup (once)**

* Bucket name: `react-devsecops-pipeline`  
* Enable **Static Website Hosting**  
* Disable block public access  
* Attach bucket policy

---

## **GitHub Secrets**

Add:
```
AWS_ACCESS_KEY_ID  
AWS_SECRET_ACCESS_KEY  
AWS_REGION  
S3_BUCKET
```
---

## **CD Workflow**

📄 `.github/workflows/cd.yml`
```
name: CD Pipeline

on:  
  push:  
    branches: [ main ]

jobs:  
  deploy:  
    runs-on: ubuntu-latest

    steps:  
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4  
        with:  
          node-version: 18

      - run: npm ci  
      - run: npm run build

      - name: Upload to S3  
        uses: jakejarvis/s3-sync-action@v0.5.1  
        with:  
          args: --delete  
        env:  
          AWS_S3_BUCKET: ${{ secrets.S3_BUCKET }}  
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}  
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}  
          AWS_REGION: ${{ secrets.AWS_REGION }}  
          SOURCE_DIR: build
```
---

# **FINAL FLOW** 
```
Local Dev  
 └─ git init → commit → tag  
 └─ bugfix branch → merge → tag  
 └─ push to GitHub

CI  
 └─ PR triggers build + test + lint + security  
 └─ Auto labels + code review

CD  
 └─ Merge to main  
 └─ Build artifact  
 └─ Deploy static files to S3

CR Changes  
 └─ Same CI  
 └─ Merge → CD again
```
---

# **🔐 PART 1: Add Security to CI (Shift-Left Security)**

We’ll add **SAST, SCA, and DAST** in the **CI pipeline**, so every PR is validated **before merge**.

---

## **1️⃣ SAST – SonarCloud (Static Application Security Testing)**

### **What it does**

* Scans **source code**  
* Finds **bugs, vulnerabilities, code smells**  
* Enforces **quality gates** (fail PR if not met)

### **Setup (one time)**

1. Create SonarCloud account  
2. Link GitHub repo  
3. Generate **SONAR_TOKEN**

Add GitHub secret:
```
SONAR_TOKEN
```
---

### **Update CI workflow (add SAST)**

📄 `.github/workflows/ci.yml` (add step)
```
     - name: SonarCloud Scan

        uses: SonarSource/sonarcloud-github-action@v2

        env:

          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

        with:

          args: >

            -Dsonar.projectKey=your-org_react-devsecops-pipeline

            -Dsonar.organization=your-org

            -Dsonar.sources=src
```
✅ Result:

* PR shows **SonarCloud status**  
* Code quality gate enforced

---

## **2️⃣ SCA – Snyk (Dependency & Container Security)**

### **What it does**

* Scans **npm dependencies**  
* Detects **known CVEs**  
* Prevents vulnerable libraries from merging

### **Setup**

1. Create Snyk account  
2. Connect GitHub  
3. Generate **SNYK_TOKEN**

Add secret:

SNYK_TOKEN

---

### **Add Snyk scan to CI**

     - name: Snyk Dependency Scan

        uses: snyk/actions/node@master

        env:

          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

        with:

          command: test

📌 Best practice:

* Fail build only on **high/critical**  
* Allow monitoring mode initially

---

## **3️⃣ DAST – OWASP ZAP (Dynamic Security Testing)**

DAST runs **after deployment**, against the **live app**.

We’ll trigger it **post-CD** or against a **preview environment**.

---

### **DAST workflow (separate job)**

📄 `.github/workflows/dast.yml`

name: DAST Scan

on:

  workflow_run:

    workflows: ["CD Pipeline"]

    types:

      - completed

jobs:

  zap-scan:

    runs-on: ubuntu-latest

    steps:

      - name: OWASP ZAP Scan

        uses: zaproxy/action-full-scan@v0.9.0

        with:

          target: https://your-cloudfront-url

✅ Scans:

* XSS  
* SQL injection  
* Insecure headers  
* Auth issues

---

# **🌍 PART 2: Production-grade Delivery with CloudFront + HTTPS**

Right now you deploy to **S3 static hosting**.  
We’ll upgrade it to **secure global delivery**.

---

## **4️⃣ CloudFront + S3 Architecture**

User

 ↓ HTTPS

CloudFront (CDN + TLS)

 ↓

S3 Bucket (Private)

### **Benefits**

* HTTPS (TLS)  
* Global caching  
* Faster loads  
* DDoS protection (AWS Shield)  
* No public S3 access

---

## **5️⃣ S3 Bucket (Best Practice)**

* ❌ Disable public access  
* ✅ Allow access **only via CloudFront**  
* Enable **versioning**

Bucket policy (example):
```
{

  "Version": "2012-10-17",

  "Statement": [{

    "Effect": "Allow",

    "Principal": {

      "Service": "cloudfront.amazonaws.com"

    },

    "Action": "s3:GetObject",

    "Resource": "arn:aws:s3:::react-devsecops-pipeline/*"

  }]

}
```
---

## **6️⃣ CloudFront Setup**

1. Create CloudFront distribution  
2. Origin → S3 bucket  
3. Viewer protocol → **Redirect HTTP to HTTPS**  
4. Default root object → `index.html`

---

## **7️⃣ HTTPS with ACM (Free SSL)**

* Request certificate in **us-east-1**  
* Attach to CloudFront  
* Optional: custom domain  
  `www.react-demo.com`

---

## **8️⃣ Update CD to Invalidate Cache**

Add to `cd.yml`:

     - name: Invalidate CloudFront Cache

        uses: chetan/invalidate-cloudfront-action@v2

        env:

          DISTRIBUTION: ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }}

          AWS_REGION: ${{ secrets.AWS_REGION }}

          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}

          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

          PATHS: "/*"

Add secret:
```
CLOUDFRONT_DISTRIBUTION_ID
```
---

# **🔁 FINAL ENTERPRISE FLOW (This is GOLD)**
```
Local Dev

 └─ Feature / Bugfix / CR branches

 └─ Clean commits + tags

Pull Request (CI)

 └─ Build

 └─ Unit Tests

 └─ ESLint (Static)

 └─ SonarCloud (SAST)

 └─ Snyk (SCA)

 └─ Auto Labeling

 └─ Code Review

Merge to main

 └─ Build artifact

 └─ Deploy to S3

 └─ CloudFront CDN

 └─ HTTPS

 └─ Cache invalidation

Post-Deploy

 └─ OWASP ZAP (DAST)
```
# **1️⃣ 🔐 Secret Scanning (GitHub Advanced Security)**

## **What it does**

* Detects **hardcoded secrets** (AWS keys, tokens, passwords)  
* Blocks leaks **before merge**  
* Works on **commits + PRs**

---

## **Enable (Repo Settings – once)**

**GitHub → Settings → Security & analysis**  
Enable:

* ✅ **Secret scanning**  
* ✅ **Push protection**  
* ✅ Dependency graph (already used by Snyk)

📌 This is **platform-level**, not YAML-based.

---

## **Result**

If someone commits this ❌:

`const AWS_KEY = "AKIAxxxxxxxxxxxxx";`

GitHub will:

* Block the push  
* Show secret type  
* Require explicit override

👉 **No pipeline bypass** → security by default

---

# **2️⃣ 🚦 Quality Gates Blocking Merge (Hard Enforcement)**

We’ll block merge if:

* Tests fail  
* SonarCloud quality gate fails  
* Snyk finds high/critical vulns

---

## **A. Enforce SonarCloud Quality Gate**

### **Update Sonar step (CI)**

     - name: SonarCloud Scan

        uses: SonarSource/sonarcloud-github-action@v2

        env:

          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

Now configure **in SonarCloud UI**:

* Coverage ≥ 80%  
* No new critical issues  
* No new security hotspots

❌ If gate fails → PR blocked

---

## **B. Fail pipeline on Snyk High/Critical**

     - name: Snyk Scan (Fail on High)

        uses: snyk/actions/node@master

        env:

          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

        with:

          command: test

          args: --severity-threshold=high

---

## **C. Protect main branch**

**GitHub → Settings → Branch protection**

* Require CI checks to pass  
* Require PR review  
* Block direct push to `main`

---

## **Result**

PR → CI

   ├─ Tests ❌ → Merge blocked

   ├─ SAST ❌ → Merge blocked

   ├─ SCA ❌ → Merge blocked

   └─ Review ❌ → Merge blocked

---

# **3️⃣ 🌱 Preview Environments per Pull Request**

Each PR gets:

* Its **own deployed URL**  
* Automatically **created & destroyed**

---

## **Strategy (Best Practice)**

* Use **S3 prefix per PR**  
* Deploy to:

s3://react-devsecops-pipeline/pr-<PR_NUMBER>/

---

## **Preview Deployment Workflow**

📄 `.github/workflows/preview.yml`

name: PR Preview Environment

on:

  pull_request:

    types: [opened, synchronize, reopened, closed]

jobs:

  preview:

    if: github.event.action != 'closed'

    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4

        with:

          node-version: 18

      - run: npm ci

      - run: npm run build

      - name: Deploy Preview to S3

        uses: jakejarvis/s3-sync-action@v0.5.1

        with:

          args: --delete

        env:

          AWS_S3_BUCKET: ${{ secrets.S3_BUCKET }}

          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}

          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

          AWS_REGION: ${{ secrets.AWS_REGION }}

          SOURCE_DIR: build

          DEST_DIR: pr-${{ github.event.pull_request.number }}

---

## **Cleanup on PR Close**

 cleanup:

    if: github.event.action == 'closed'

    runs-on: ubuntu-latest

    steps:

      - name: Delete Preview Environment

        run: |

          aws s3 rm s3://${{ secrets.S3_BUCKET }}/pr-${{ github.event.pull_request.number }} --recursive

---

## **Result**

PR comment:

Preview URL:

https://cdn-domain/pr-42/index.html

---


# **4️⃣ 📦 Artifact Versioning Using Git Tags (Release Discipline)**

We’ll:

* Use **Git tags as the source of truth**  
* Version artifacts  
* Deploy only **tagged releases**

---

## **Tagging Flow (Local)**
```
git tag -a v1.2.0 -m "Release v1.2.0"

git push origin v1.2.0
```
---

## **CD Triggered by Tags Only**

📄 Update `.github/workflows/cd.yml`

on:

  push:

    tags:

      - "v*"

---

## **Versioned Artifact Upload**

     - name: Upload Versioned Build

        uses: jakejarvis/s3-sync-action@v0.5.1

        env:

          AWS_S3_BUCKET: ${{ secrets.S3_BUCKET }}

          SOURCE_DIR: build

          DEST_DIR: releases/${{ github.ref_name }}

---

## **Final Deployment**

After validation:

`DEST_DIR: prod`

---

## **Resulting Structure**
```
S3 Bucket

├─ releases/

│  ├─ v1.0.0/

│  ├─ v1.1.0/

│  └─ v1.2.0/

└─ prod/
```
Rollback = one command:

`aws s3 sync s3://bucket/releases/v1.1.0 s3://bucket/prod`

# **🏆 FINAL ENTERPRISE FLOW (COMPLETE)**
```
Developer

 └─ Feature / Bugfix / CR branch

 └─ Clean commits

Pull Request

 ├─ Secret scanning

 ├─ Build + tests

 ├─ ESLint

 ├─ SonarCloud (SAST)

 ├─ Snyk (SCA)

 ├─ Preview environment

 ├─ Auto-labeling

 └─ Quality gate enforcement

Merge to main (protected)

 └─ Tag-based release

CD

 ├─ Build artifact

 ├─ Versioned storage

 ├─ Deploy to S3

 ├─ CloudFront CDN

 ├─ HTTPS

 ├─ Cache invalidation

 └─ OWASP ZAP (DAST)

Rollback

 └─ Redeploy previous tag
```
---

# **🏗️ 1️⃣ SYSTEM ARCHITECTURE DIAGRAM (Production)**

### **React Static Website – Secure & Scalable**
```
┌──────────────┐  
│   End User   │  
└──────┬───────┘  
       │ HTTPS (443)  
         				    ▼  
┌───────────────────────┐  
│   Amazon CloudFront   │  
│  - CDN (Global Edge)  │  
│  - HTTPS (ACM Cert)   │  
│  - DDoS Protection    │  
└────────┬──────────────┘  
         │ Private Access  
    ▼  
┌───────────────────────┐  
│     Amazon S3         │  
│  - Static React App   │  
│  - Versioned Artifacts│  
│  - No Public Access   │  
└───────────────────────┘
```
### **Key Architecture Decisions**

* ✅ **S3 private bucket** (no public access)  
* ✅ **CloudFront-only access** to S3  
* ✅ **HTTPS via ACM**  
* ✅ **Versioned releases for rollback**

---

# **🔁 2️⃣ CI/CD PIPELINE DIAGRAM (DevSecOps)**

### **End-to-End Pipeline (Local → Prod)**
```
**Developer Laptop**  
┌───────────────────────────────┐  
│  git init / commit / tag      │  
│  feature / bugfix / CR branch │  
└───────────────┬───────────────┘  
                │ push / PR  
                ▼  
┌──────────────────────────────────────────┐  
│               GitHub                     │  
│  - Protected main branch                │  
│  - Auto labeling                        │  
│  - Secret scanning                      │  
└───────────────┬──────────────────────────┘  
                │ Pull Request  
                ▼  
┌──────────────────────────────────────────┐  
│               CI PIPELINE                │  
│  ✔ npm install                           │  
│  ✔ Unit tests                            │  
│  ✔ ESLint (Static Analysis)              │  
│  ✔ SonarCloud (SAST)                     │  
│  ✔ Snyk (SCA)                            │  
│  ✔ Quality Gates                         │  
│  ✔ Preview Environment (PR)              │  
└───────────────┬──────────────────────────┘  
                │ Merge Approved  
                ▼  
┌──────────────────────────────────────────┐  
│         TAG-BASED RELEASE (vX.Y.Z)       │  
└───────────────┬──────────────────────────┘  
                │  
                ▼  
┌──────────────────────────────────────────┐  
│               CD PIPELINE                │  
│  ✔ Build React App                       │  
│  ✔ Store versioned artifact              │  
│  ✔ Deploy to S3                          │  
│  ✔ CloudFront cache invalidation         │  
└───────────────┬──────────────────────────┘

                │  
                ▼  
┌──────────────────────────────────────────┐  
│          POST-DEPLOY SECURITY            │  
│  ✔ OWASP ZAP (DAST)                      │  
└──────────────────────────────────────────┘
```
---

# **🌱 3️⃣ PREVIEW ENVIRONMENT FLOW (PR-Level)**
```
Pull Request #42  
        │  
        ▼  
CI Build  
        │  
        ▼  
S3 Bucket  
┌─────────────────────────────┐  
│     /pr-42/index.html       │  
│     /pr-42/static/*         │  
└──────────────┬──────────────┘  
               │  
               ▼  
CloudFront URL  
https://cdn-domain/pr-42/
```
### **On PR Close**

PR Closed → Preview deleted automatically

---

# 

# **📦 4️⃣ ARTIFACT VERSIONING & ROLLBACK**
```
S3 Bucket Structure  
┌───────────────────────────────┐  
│ releases/                     │  
│   ├─ v1.0.0/                  │  
│   ├─ v1.1.0/                  │  
│   └─ v1.2.0/                  │  
│ prod/  → current stable       │  
└───────────────────────────────┘
```
### **Rollback (Zero Downtime)**

Copy releases/v1.1.0 → prod/  
Invalidate CloudFront cache

---

# **🔐 5️⃣ SECURITY LAYERS (Shift-Left Model)**
```
Code Commit  
   │  
   ▼  
Secret Scanning (GitHub)  
   │  
   ▼  
SAST (SonarCloud)  
   │  
   ▼  
SCA (Snyk)  
   │  
   ▼  
DAST (OWASP ZAP)
```
---

