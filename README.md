```markdown
# Static Site Deployment on AWS S3 & CloudFront

![Static Site Deployment Architecture](https://d1.awsstatic.com/Projects/P4105033/architecture-Diagram.427a8e0e5fe83a7a78f7e4d8c1e0b1c5e7a1a0a8.png)

A complete guide and template for deploying blazing-fast static websites on AWS infrastructure using S3 for storage and CloudFront as a global CDN.

## Features

- 🚀 **High Performance**: Global content delivery via CloudFront's CDN
- 💰 **Cost Effective**: Pay only for what you use (often <$1/month for small sites)
- 🔒 **Secure**: Free SSL certificates via AWS Certificate Manager
- ⚡ **Scalable**: Automatically handles traffic spikes
- 🌍 **Global Availability**: 200+ edge locations worldwide

## Prerequisites

- AWS account (Free tier eligible)
- Static website files (HTML, CSS, JS, images)
- Domain name (optional but recommended)
- AWS CLI installed (optional but helpful)

## Step-by-Step Deployment Guide

### 1. Prepare Your Static Site

Ensure your site works locally and all links are relative (not absolute).

```bash
# Example structure
your-site/
├── index.html
├── css/
│   └── style.css
├── js/
│   └── script.js
└── images/
    └── logo.png
```

### 2. Create an S3 Bucket

1. Navigate to [AWS S3 Console](https://s3.console.aws.amazon.com/)
2. Click "Create bucket"
3. Enter a globally unique name (e.g., `www.yourdomain.com`)
4. Select region closest to your users
5. Uncheck "Block all public access" and acknowledge
6. Click "Create bucket"

### 3. Enable Static Website Hosting

1. Select your bucket in S3 console
2. Go to "Properties" tab
3. Scroll to "Static website hosting"
4. Click "Edit" and select "Enable"
5. Specify index document (typically `index.html`)
6. Click "Save changes"

### 4. Set Bucket Policy for Public Access

1. Go to "Permissions" tab
2. Click "Bucket Policy"
3. Add the following policy (replace bucket name):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::www.yourdomain.com/*"
        }
    ]
}
```

### 5. Upload Your Files

You can upload manually via the AWS Console or use AWS CLI:

```bash
# Install AWS CLI if needed
aws configure  # Set up your credentials

# Sync local files to S3
aws s3 sync ./your-site s3://www.yourdomain.com --delete
```

### 6. Set Up CloudFront Distribution

1. Navigate to [CloudFront Console](https://console.aws.amazon.com/cloudfront/)
2. Click "Create distribution"
3. Under Origin, select your S3 bucket
4. Important: Select "Yes use OAI" and create new OAI
5. Under "Default cache behavior", choose "Redirect HTTP to HTTPS"
6. Under "Settings", specify your domain name (if you have one)
7. Click "Create distribution" (Deployment takes ~15 minutes)

### 7. Configure Custom Domain (Optional)

1. Request a certificate in [AWS Certificate Manager](https://console.aws.amazon.com/acm/)
2. Add your domain (e.g., `yourdomain.com` and `www.yourdomain.com`)
3. Verify ownership via DNS or email
4. Update your CloudFront distribution to use this certificate

### 8. Set Up DNS (Route 53 or other provider)

For Route 53:
1. Create a new hosted zone for your domain
2. Create an A record (Alias) pointing to your CloudFront distribution
3. For www subdomain, create CNAME or another A record

For other providers:
- Create CNAME record pointing to your CloudFront domain (e.g., `d123.cloudfront.net`)

## Automated Deployment (CI/CD)

Set up automatic deployments when you push to GitHub:

1. Create a new IAM user with S3 and CloudFront permissions
2. Add secrets to your GitHub repo:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`
   - `AWS_S3_BUCKET`
   - `AWS_CLOUDFRONT_DISTRIBUTION_ID`

3. Add this GitHub Action workflow (`.github/workflows/deploy.yml`):

```yaml
name: Deploy to AWS

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Deploy to S3
        run: |
          aws s3 sync ./ s3://${{ secrets.AWS_S3_BUCKET }} --delete
          
      - name: Invalidate CloudFront Cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.AWS_CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
```

## Cost Optimization Tips

1. **S3 Storage**: $0.023 per GB/month (first 50TB)
2. **CloudFront**: 
   - $0.085 per GB data transfer out (first 10TB)
   - $0.010 per 10,000 HTTPS requests
3. **Free Tier**:
   - 1 TB of CloudFront data transfer out
   - 2 million CloudFront requests
   - 5 GB of S3 storage

Estimated costs for small sites: <$1/month

## Troubleshooting

**Issue**: 403 Forbidden when accessing site
- ✅ Verify bucket policy allows public read access
- ✅ Check if objects are publicly accessible in S3

**Issue**: Changes not appearing
- ✅ CloudFront cache takes time to invalidate (default 24h)
- ✅ Manually invalidate cache in CloudFront console

**Issue**: Mixed content warnings
- ✅ Ensure all resources use HTTPS URLs
- ✅ Set up proper redirects from HTTP to HTTPS

## Advanced Configuration

### Custom Error Pages
1. In S3 static hosting settings, specify error document (e.g., `404.html`)
2. In CloudFront, create custom error responses

### URL Rewrites for SPAs
Configure CloudFront to return `index.html` for all paths:

1. Go to your distribution
2. Under "Error pages", create custom error response
3. HTTP error code: 403 (or 404)
4. Customize error response: Yes
5. Response page path: `/index.html`
6. HTTP response code: 200

## License

MIT

```

## Key Features of This README:

1. **Comprehensive Step-by-Step Guide**:
   - From S3 bucket creation to CloudFront configuration
   - Includes both console and CLI methods

2. **Visual Architecture Diagram**:
   - Shows the relationship between components

3. **CI/CD Automation**:
   - Ready-to-use GitHub Actions workflow

4. **Cost Optimization**:
   - Clear pricing breakdown and free tier information

5. **Troubleshooting Section**:
   - Solutions for common issues

6. **Advanced Configurations**:
   - SPA routing, custom error pages, etc.

7. **Multiple Deployment Methods**:
   - Manual uploads and automated CI/CD
