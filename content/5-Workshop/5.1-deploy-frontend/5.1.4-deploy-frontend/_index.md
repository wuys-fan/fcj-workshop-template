---
title: "Deploy Frontend to S3 & CloudFront"
weight: 4
chapter: false
pre: " <b> 5.1.4 </b> "
---

# Deploy Frontend to S3 & CloudFront

In alignment with our designed architecture, we will not use pre-packaged deployment services. Instead, we will deploy the static frontend to **Amazon S3**, use **Amazon CloudFront** as a CDN for global acceleration, and integrate **AWS WAF** for security.

## 1. Build the Source Code (Local)

Before deploying to AWS, we need to package (build) the ReactJS source code into static files (HTML, CSS, JS).

1. Open the Terminal in your `frontend` directory:
```bash
cd frontend
npm run build
```
2. Once completed, a `build` (or `dist`) folder will be generated. This is the folder we will upload to AWS.

![ReactJS Build Output](/images/1-Worklog/react_build_output.png)

---

## 2. Create and Configure Amazon S3 Bucket

### 2.1 Create a Bucket
1. Go to AWS Console → Search for **S3**
2. Click **Create bucket**
3. **Bucket name**: `pet-resort-frontend-prod` (Must be globally unique)
4. **AWS Region**: `ap-southeast-1`
5. Uncheck **Block all public access** (Since this is a public website).
6. Click **Create bucket**.

![Create S3 Bucket](/images/5-Workshop/s3-create.png)

### 2.2 Upload Source Code and Enable Website Hosting
1. Open the newly created Bucket.
2. Click **Upload** → Drag and drop all contents from the `build` (or `dist`) folder created in Step 1 → Click **Upload**.
3. Switch to the **Properties** tab.
4. Scroll down to **Static website hosting** → Click **Edit**.
5. Select **Enable**.
6. Index document: `index.html`.
7. Click **Save changes**.

{{% notice info %}}
📝 **Note**: You must grant public read access via Bucket Policy so users can view the content.
{{% /notice %}}

---

## 3. Configure Amazon CloudFront & WAF

To make the web application faster and more secure, we place CloudFront in front of the S3 bucket.

### 3.1 Create a Distribution
1. Go to AWS Console → Search for **CloudFront** → **Create Distribution**.
2. **Origin domain**: Paste the **S3 Static Website Hosting Endpoint** (e.g., `pets-hop-frontend.s3-website-ap-southeast-1.amazonaws.com`) of the bucket you just created.
   * *Note*: Do not select the default auto-suggested bucket API endpoint, as it may cause permission/routing issues.
3. **Viewer protocol policy**: Select **Redirect HTTP to HTTPS** (Mandatory for security).
4. **Allowed HTTP methods**: Select **GET, HEAD**.
5. **Cache policy**: Choose **CachingOptimized**.
6. **Web Application Firewall (WAF)**: Select **Enable security protections** to automatically block malicious requests.
7. Click **Create distribution**.

![Create CloudFront Distribution](/images/5-Workshop/cloudfront-create.png)

### 3.2 Get the Access URL
After 3-5 minutes, CloudFront will finish deploying.
Copy the URL under **Distribution domain name** (e.g., `d1x2y3z4.cloudfront.net`) and paste it into your browser to test the website.

---

## 4. Automated CI/CD with GitHub Actions (Optional)

To avoid manual uploads every time the code is modified, we set up **GitHub Actions** to automatically push code to S3.

Create a `.github/workflows/deploy.yml` file in your repository:

```yaml
name: Deploy Frontend to S3
on:
  push:
    branches: [ main ]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Build
      run: |
        cd frontend
        npm install
        npm run build
    - name: Deploy to S3
      run: aws s3 sync frontend/build/ s3://pet-resort-frontend-prod --delete
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## 5. Troubleshooting

### Issue 1: 403 Access Denied Error
**Solution**: Check your S3 Bucket Policy. Ensure you have added a policy allowing `s3:GetObject` from all sources (`*`) or configured CloudFront OAC correctly.

### Issue 2: 404 Error on Refresh or Direct Link
Since ReactJS is a Single Page Application (SPA), AWS S3 does not understand virtual routes.
**Solution**: 
1. Go to CloudFront → Select your Distribution → **Error pages** tab.
2. Click **Create custom error response**.
3. HTTP error code: `404: Not Found`.
4. Customize error response: `Yes`.
5. Response page path: `/index.html`.
6. HTTP Response code: `200: OK`.

![CloudFront Error Pages](/images/5-Workshop/cloudfront-error.png)

---

## Next Steps

Continue with [Application Testing](../5.1.5-testing/).