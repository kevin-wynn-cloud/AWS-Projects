# Creating My Static Website

# Step 1: Register a Custom Domain

I registered the custom domain kevin-wynn.com using Route53.

![1](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/786c63ed-6259-4eda-b767-6062b4f7f8e6)

# Step 2: Creating S3 Buckets

I created several S3 buckets to organize my website content:

- A bucket for my root domain (kevin-wynn.com)
- A bucket for the subdomain (www.kevin-wynn.com)
- A bucket for logs
- A test bucket for private testing of new site variations.

![2](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/518d0067-e335-4c83-918d-afaf3335979f)

# Step 3: Changing Bucket Properties

I enabled static website hosting for kevin-wynn.com and uploaded the index.html and asset files (CSS, JavaScript, images, and fonts) to the kevin-wynn.com bucket.

![3](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/8b85b894-a00d-4beb-b005-489af8dcc800)

# Step 4: Configuring Subdomain

I configured the www.kevin-wynn.com bucket by enabling static website hosting and setting up redirection for requests to kevin-wynn.com. I chose HTTP, as I planned to use CloudFront for HTTPS.

![4](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/ba5fdfde-62dd-4935-846c-88c7f34320d1)

# Step 5: Route 53 Records

I added A records in Route 53 for both the root domain and the subdomain, pointing them to the respective S3 buckets. I used an alias to direct traffic to the S3 website endpoint. I also created a wildcard record for *.kevin-wynn.com.

![5](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/e76ad534-35cf-414f-b601-5d1c8424271e)

# Step 6: AWS Certificate Manager

I went to AWS Certificate Manager (ACM), ensuring I was in the N. Virginia region (required for CloudFront). I requested a certificate for my domain and subdomain using a wildcard and validated it through DNS (using Route 53) due to owning the domain.

![6](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/72e4445b-2273-4acc-a76d-2f5f2d93c61d)

# Step 7: Adding CNAME Records in Route 53

In Route 53, I added CNAME records to enable AWS Certificate Manager to validate and issue my SSL certificate.

![7](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/e9ebe99e-bb93-4cc4-80d1-f3dd44205382)

# Step 8: Creating CloudFront Distribution

After switching to the global region, I went to CloudFront and set up my root domain bucket as the CloudFront origin. I configured an origin access identity for added security and to block public access to my S3 bucket while serving its content. I also set up HTTP to HTTPS redirection and included the *.kevin-wynn.com alternate domain name.

![8](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/e2fbf71b-c440-4881-be8a-cb5e443c210b)

# Step 9: Updating Bucket Policy for CloudFront

CloudFront generated a new policy to allow it to access my S3 bucket while preventing public access. I applied this policy to my kevin-wynn.com bucket.

```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCloudFrontAccess",
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudfront.amazonaws.com"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::kevin-wynn.com/*",
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": "CloudFront distribution ID"
                }
            }
        }
    ]
}
```

# Step 10: Updating Route 53 Alias Records

In Route 53, I updated my records to point to the CloudFront distribution instead of the alias S3 website endpoint.

![10](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/7a08dbe7-3978-4164-a815-f0e575437245)

# Step 11: Testing My Website

I tested my website by entering kevin-wynn.com into the address bar and ensured that the content loaded with HTTPS. I also checked other subdomains like www.kevin-wynn.com and net.kevin-wynn.com to confirm proper functionality.

![11](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/231abef3-d207-4c39-b357-76f8ff795627)

