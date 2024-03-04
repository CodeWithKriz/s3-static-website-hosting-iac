# S3 Static Website Hosting - IAC
Infrastructure as Code using AWS Cloudformation template to manage AWS S3 static website hosting with ACM, Route53, Record Sets, Cloudfront, CloudfrontOAC, and naked &amp; subdomain S3 buckets

![s3-static-website-hosting-subdomain](https://github.com/CodeWithKriz/s3-static-website-hosting-iac/assets/66562899/7fa71124-b0c4-4a76-9b7c-43f7a2c70c9f)

---

# create-acm-certificate.yaml
## Description:
Create ACM Certificate and output its ARN.

## Instruction:
1. This stack can be used to request ACM SSL certificate in the two ways
  * ACM w/ DNS validation - provide Route 53 "HostedZoneId"
  * ACM w/ EMAIL validation - provide Email Domain Name (must be same as Domain Name in most cases)

## Usage:
### ACM w/ DNS validation
* DomainName - example.com
* ValidationMethod - DNS
* HostedZoneId - <Route53_Hosted_Zone_Id>

### Outcome
1. Requests new ACM using DNS validation

### ACM w/ EMAIL validation
* DomainName - example.com
* ValidationMethod - EMAIL
* ValidationDomain - example.com

### Outcome
1. Requests new ACM using EMAIL validation

## Outputs:
1. Outputs.Arn

---

# s3-static-website-hosting-subdomain.yaml
## Description:
Cloudformation template to manage aws services required for aws s3 static website hosting.
During execution, manually copy Route 53 NS to Domain Registrar (if domain registered externally) if stack creates Route 53.

## Instruction:
1. This stack can be used to register domains in the following ways
  * www domain only, creates domain without naked domain redirection
  * www & naked (If "RedirectNakedDomain" set to "Yes") domain together, redirects naked domain to www domain
  * other subdomains individually (like "dev" or "test") w/o naked domain redirection
ACM certificate can be requested in two ways
  * If new certificate is required, pass the template URL "CreateAcmTemplateUrl" Outputs "Arn"
  * If certificate already exists, pass the "AcmArn"
3. If any of these services are already created just provide their Id or ARN
  * AcmArn
  * Route53HostedZoneId
  * CloudfrontOACId

## Usage:
### www domain registration only
* AcmArn - <ACM_ARN> (or) "" (create new one)
* CloudfrontOACId - <Cloudfront_OAC_Id> (or) "" (create new one)
* CreateAcmTemplateUrl - https://${s3-bucket}.s3.amazonaws.com/path/to/create-acm-stack.yaml (if "AcmArn" empty)
* DomainName - example.com
* RedirectNakedDomain - No
* Route53HostedZoneId - <Route53_Hosted_Zone_Id> (or) "" (create new one)
* SubDomain - www

### Outcome
1. Creates www s3 bucket
2. If "Route53HostedZoneId" is empty, creates new Hosted Zone
3. If "AcmArn" is empty, requests new ACM using "CreateAcmTemplateUrl"
4. If "CloudfrontOACId" is empty, creates new Origin Access Control
5. Creates www Cloudfront distribution
6. Adds www Cloudfront Recordset to Route 53 Hosted Zone
7. Adds S3 bucket policy to allow request only from Cloudfront distribution

### www domain registration & naked domain redirection
* AcmArn - <ACM_ARN> (or) "" (create new one)
* CloudfrontOACId - <Cloudfront_OAC_Id> (or) "" (create new one)
* CreateAcmTemplateUrl - https://${s3-bucket}.s3.amazonaws.com/path/to/create-acm-stack.yaml (if "AcmArn" empty)
* DomainName - example.com
* RedirectNakedDomain - Yes
* Route53HostedZoneId - <Route53_Hosted_Zone_Id> (or) "" (create new one)
* SubDomain - www

### Outcome
1. Creates www s3 bucket
2. Creates naked s3 bucket and redirects to www domain
3. If "Route53HostedZoneId" is empty, creates new Hosted Zone
4. If "AcmArn" is empty, requests new ACM using "CreateAcmTemplateUrl"
5. If "CloudfrontOACId" is empty, creates new Origin Access Control
6. Creates www Cloudfront distribution
7. Creates naked Cloudfront distribution
8. Adds www Cloudfront Recordset to Route 53 Hosted Zone
9. Adds naked Cloudfront Recordset to Route 53 Hosted Zone
10. Adds S3 bucket policy to allow request only from Cloudfront distribution

### sub domain registration
* AcmArn - <ACM_ARN> (or) "" (create new one)
* CloudfrontOACId - <Cloudfront_OAC_Id> (or) "" (create new one)
* CreateAcmTemplateUrl - https://${s3-bucket}.s3.amazonaws.com/path/to/create-acm-stack.yaml (if "AcmArn" empty)
* DomainName - example.com
* RedirectNakedDomain - No
* Route53HostedZoneId - <Route53_Hosted_Zone_Id> (or) "" (create new one)
* SubDomain - dev

### Outcome
1. Creates subdomain s3 bucket
2. Creates subdomain Cloudfront distribution
3. Adds subdomain Cloudfront Recordset to Route 53 Hosted Zone
4. Adds S3 bucket policy to allow request only from Cloudfront distribution

## Outputs:
* AcmArn
* SubDomainS3Bucket
* NakedDomainS3Bucket
* Route53HostedZoneId
* CloudfrontOACId
* SubDomainCloudfrontDistribution
* NakedDomainCloudfrontDistribution

## Notes:
1. During execution, manually copy Route 53 Nameservers to Domain Registrar (if domain registered externally) if stack creates Route 53
2. Add appropriate "Tags" while executing the stack. These tags will be automatically tagged to the services. Like
  * Environment - <Environment>
  * awsApplication - <My_Application_Arn>
