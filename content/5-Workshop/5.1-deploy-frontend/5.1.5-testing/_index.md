---
title: "Testing and Verification"
weight: 5
chapter: false
pre: " <b> 5.1.5 </b> "
---

# Testing and Verification

## 1. Functionality Check

### 1.1 Check all pages

- ✅ Pet Resort homepage loads correctly
- ✅ Menu displays Products & Spa Services
- ✅ Login / Booking forms render properly
- ✅ Navigation bar works
- ✅ Footer is visible

### 1.2 Routing Check

Check the following URLs (replace with your CloudFront Domain):
```text
[https://d3uvhesft661gl.cloudfront.net/](https://d3uvhesft661gl.cloudfront.net/)
[https://d3uvhesft661gl.cloudfront.net/services/tam-spa-thu-cung](https://d3uvhesft661gl.cloudfront.net/services/tam-spa-thu-cung)
[https://d3uvhesft661gl.cloudfront.net/login](https://d3uvhesft661gl.cloudfront.net/login)
```

Refresh each sub-page → No 404 error ✅

## 2. CI/CD Check 

### 2.1 Verify Auto-Deploy

1. Make a minor text change on the homepage locally
2. Commit and push the code to the `main` branch via Github Desktop (or Terminal)
3. Wait for the Success status, then refresh the website to see the changes

## 3. Clean up (Optional)

If you are just practicing and want to delete resources to avoid charges:

1. Go to AWS Console → **CloudFront** → Disable the Distribution and Delete it.
2. Go to AWS Console → **S3** → Empty the Bucket (delete all files) and Delete the Bucket.
3. Confirm the deletion.