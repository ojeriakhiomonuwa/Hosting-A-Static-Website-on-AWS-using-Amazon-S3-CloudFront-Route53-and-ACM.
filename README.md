# Hosting A Static Website on AWS using Amazon S3, CloudFront, Route53 and ACM.
![](images/Untitled%20Diagram.drawio%20(1).png)

Hello Cloud Enthusiasts, In my previous [article](https://github.com/ojeriakhiomonuwa/host-a-static-website-on-AWS-using-S3-buckets.git), I explained in detail how to host a static website on AWS using S3. The static website had a bucket website endpoint which is long and is difficult for users to remember, also it is not secured. In this article, I will explain how to host static websites using S3, CloudFront, Route53 and ACM, these will make the website secured, have a custom domain name, fast and globally available with ultra minimal latency. So, Lets dive in.

## OVERVIEW OF AWS SERVICES USED

1. **Amazon S3:** Amazon S3 (Simple Storage Service) is a cloud-based object storage service provided by Amazon Web Services (AWS). It is designed to store and retrieve any amount of data over the internet. S3 is highly scalable, durable, and secure, making it one of the most popular storage solutions for various types of data, including images, videos, documents, backups, and application data. It offers 99.999999999% (11 nines) durability for stored objects.
   
2. **CloudFront:** Amazon CloudFront is a content delivery network (CDN) service designed to deliver content, including web pages, images, videos, applications, and other static and dynamic assets, to users with low latency and high data transfer speeds. CloudFront accelerates the delivery of content by caching it in a network of globally distributed edge locations, which are geographically distributed data centers around the world.
It integrates with other AWS services like AWS Identity and Access Management (IAM) and AWS Certificate Manager (ACM) for secure access and SSL/TLS encryption.
   
3. **Route53:** Route 53 is a highly scalable and reliable Domain Name System (DNS) web service provided by Amazon Web Services (AWS). It is named after the UDP port 53, which is used for DNS requests. Route 53 helps businesses and developers manage and route internet traffic for domain names and web applications with high availability, low latency, and robust DNS management capabilities.
   
4. **Amazon Certificate Manager (ACM):** AWS Certificate ManagerACM allows you to request public SSL/TLS (Secure Sockets Layer/Transport Layer Security) certificates for domain names used with AWS services, such as Amazon CloudFront, Elastic Load Balancing, Amazon API Gateway, and more.  


### From our linked [article](https://github.com/ojeriakhiomonuwa/host-a-static-website-on-AWS-using-S3-buckets.git), already hosted our static website using S3 buckets, now we are going to configure route 53 for DNS management.  
Prior to configuring Route53 for DNS management, We need to purchase a custom domain name from a domain registrar, you can buy a domain name on [Namecheap](https://www.namecheap.com/?gad_source=1&gad_campaignid=11301910042&gbraid=0AAAAADzFe20In8qs6XGEOZVPy4rnU8LB3&gclid=Cj0KCQjw-4XFBhCBARIsAAdNOkuJeTy14iGZeKkYNhMTZAQlr4sz35TfIU_7qiMxUcBPY12tTI4V7AYaAuGLEALw_wcB), or [Godaddy](https://www.godaddy.com/en-ph/offers/domain-airo?isc=sem3year&countryview=1&currencyType=USD&cdtl=c_22730426281.g_190146637388.k_kwd-350064496.a_764234124367.d_c.ctv_g&bnb=b&gad_source=1&gad_campaignid=22730426281&gbraid=0AAAAAD_AGdyYUzari23CY6-Zm5k2hnLDB&gclid=Cj0KCQjw-4XFBhCBARIsAAdNOksgqBiTjMBsu2za0DOnExrGtEgMGvoDy0rC8xUK8GgSdRy84IBPBvsaAliXEALw_wcB).   

### Next, on the AWS Console, search for Route53, click on it then click on hosted zones and create hosted zone.
![](images/1.PNG)

### Next, type the name of the custom domain you purchased on Namecheap or GoDaddy. Scroll down and click on “create hosted zone”.
![](images/2.PNG)

**If you click on your domain name under hosted zone names, and then scroll down and check Records tab, you should see two records (NS and SOA)**
![](images/3.PNG)

### Next, copy the NS (Name Servers) records.
Go to your domain registrar (e.g Namecheap or GoDaddy), under Domain List , locate your domain name, click on manage and then paste the NS records copied under “Custom DNS”. Accept changes and move to the next step.
![](images/4.PNG)

we have configured DNS management in Route53 for our domain. We will come back to complete the configuration.

### Next step is to Create a SSL/TLS Certificate for CloudFront

On the AWS console search for “AWS Certificate Manager” (ACM), click on Certificate Manager
![](images/5.PNG)

Once there, select “Request a certificate.” In the provided field, enter your custom domain name (e.g., ojes.online). Choose DNS validation as the preferred method and proceed by clicking on the “Request”.
![](images/6.PNG)

Click on “View Certificate”, then click on the certificate ID and click on “create records in route53”, click on “create records” then wait while the status change from “pending validation” to “issued".  
Once the status changes from 'pending validation' to 'issued', go back to Route53, refresh the page, you should see a new `CNAME record` and you are done with ACM.
![](images/7.PNG)

###  Next step is toConfigure CloudFront Distributions for Website Delivery.
On the AWS console, search for CloudFront. On the CloudFront dashboard, click on “create CloudFront distribution”,fill the distribution name and select  single website or app for distribution type, fill in your domain name and click check domain to make sure the domain is managed by route53.    
Click next
![](images/8.PNG)
![](images/9.PNG)

### choose amazon s3 as origin type
![](images/10.PNG)

### For origin, click on browse s3 and select the s3 buckets containing  your website files
![](images/11.PNG)

**Don't forget to Allow private S3 bucket access to CloudFront**  
Click next

### Don't enable security protections
![](images/12.PNG)

### Pick your available TLS certificate (the one issued by the ACM)
![](images/13.PNG)

### Create distribution
![](images/14.PNG)

### In your cloud front distribution settings, click on edit and add `index.html` to the default root object
![](images/addition.PNG)

**scroll to the end and you'd find the field to add `index.html` to the default root object**
![](images/Capture.PNG)

`Setting index.html as the default root object ensures that when users visit your domain (e.g., https://ojes.online/), CloudFront automatically serves index.html. Without it, requests to the root would return an error instead of your homepage.`

**Your Cloudfront distribution has been created successfully,now lets confirm your cloudfront distribution settings**    
In your cloudfront console,copy your  CloudFront distribution domain name. It looks like
`d1234abcdxyz.cloudfront.net.`
![](images/15.PNG)

paste it in your browser and your hosted website should load.
![](images/16.PNG)

### Your cloudfront distribution domain name looks ugly, so we are going to connect our domain name with our cloudfront distribution using route53.  
### With Route 53, when someone types your domain, Route 53 looks up where to send traffic and points it to CloudFront.
### we are going to create  Alias records `(A/AAAA)` that map directly to the CloudFront distribution

### 1. Go to Route 53
Open the Route 53 console.
Select your hosted zone for your domain
![](images/17.PNG)

### 2. Add the A record (Alias to CloudFront)

* Click Create record.

* In Record name, leave it blank.  
For Record type, choose A – Routes traffic to an IPv4 address and some AWS resources.

* Toggle Alias to Yes.

* Route traffic to - Alias to cloudfront distribution.

* In Alias target, pick your CloudFront distribution from the dropdown (it will show your dxxxxx.cloudfront.net).

* Click Create record.
![](images/18.PNG)

### 2. Add the AAAA record (IPv6 Support)

* Click Create record.

* In Record name, leave it blank.  
For Record type, choose AAAA – – Routes traffic to an IPv6 address and some AWS resources.

* Toggle Alias to Yes.

* Route traffic to - Alias to cloudfront distribution.

* In Alias target, pick your CloudFront distribution from the dropdown (it will show your dxxxxx.cloudfront.net).

* Click Create record.
![](images/19.PNG)

### You should have 5 records like this.
![](images/20.PNG)

### Now your domain should load your website.
![](images/21.PNG)

### Now go back to your s3 buckets and block all public access.
![](images/22.PNG)

## Congratulations on successfully hosting your static website on AWS. Using Amazon S3, CloudFront, ACM and Route53 together provide a powerful solution for Hosting, Caching, and DNS management, ensuring a reliable and cost-effective hosting experience on AWS. Thank you for following this guide until the end, and here’s to many more successful hosting endeavors! Cheers!


