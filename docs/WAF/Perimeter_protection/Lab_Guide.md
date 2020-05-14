## Create application

### OWASP Top 10 vulnerability mitigation using AWS WAF

In this section, you will use AWS WAF to mitigate a vulnerability caused by weak code on your server. The vulnerability is a Local File Inclusion (LFI) in the Broken Access Control category of OWASP Top 10 (A5 category in 2017 release). Let's suppose that this vulnerability was introduced recently by code a change, where your developer has added to your application the capability to show additional information about itself using info query string. 

1.- Test the api vulnerability in your browser. 

*	```https://[your_cloudfront_distribution_domain]/api``` : will give the same results as before.
*	```https://[your_cloudfront_distribution_domain]/api?info=about``` : will append to the response, the content of about file.
*	```https://[your_cloudfront_distribution_domain]/api?info=dependency;cat%20/etc/passwd``` : will append to the response, the content of dependency file and /etc/passwd. 

![perimeter_create_1a](/assets/images/waf/perimeter_create_1a.png)

<br/>

![perimeter_create_1b](/assets/images/waf/perimeter_create_1b.png)

2.- While waiting for your developers to correct the code and enforce access control, you will mitigate the vulnerability using AWS WAF. Go to AWS WAF Console and create a new String match condition with the below filter settings.

![perimeter_create_2a](/assets/images/waf/perimeter_create_2a.png)

<br/>

![perimeter_create_2b](/assets/images/waf/perimeter_create_2b.png)

3.- Create another filter in the same condition with the following filter.

![perimeter_create_3](/assets/images/waf/perimeter_create_3.png)

4.- Click on **Create** to finalize a condition that detects ```"/"``` and ```";"``` characters in the info query string after decoding as url.

![perimeter_create_4](/assets/images/waf/perimeter_create_4.png)

5.- Create a new rule in AWS WAF console. Choose a regular rule and reference the previously created string matching condition.

![perimeter_create_5a](/assets/images/waf/perimeter_create_5a.png)

<br/>

![perimeter_create_5b](/assets/images/waf/perimeter_create_5b.png)

6.- Finally, create a new WebACL and associate it with your CloudFront distribution. In this WebACL, you will permit all requests unless they match the rule you have just created. Confirm, review and proceed with WebAcl Creation.

![perimeter_create_6a](/assets/images/waf/perimeter_create_6a.png)

<br/>

![perimeter_create_6b](/assets/images/waf/perimeter_create_6b.png)

Click **Next** in _Create conditions_ 
 
![perimeter_create_6c](/assets/images/waf/perimeter_create_6c.png)

<br/>

![perimeter_create_6d](/assets/images/waf/perimeter_create_6d.png)

<br/>

![perimeter_create_6e](/assets/images/waf/perimeter_create_6e.png)

7.- In a few seconds, the change will propgate to AWS WAF. Test again in your browser the attacker URL. You will be blocked (403 Forbidden), and the custom error page will be returned by CloudFront.

![perimeter_create_7](/assets/images/waf/perimeter_create_7.png)

8.- Finally, go to your newly created WebACL in the AWS WAF console and check the sampled requests. It takes a couple of minutes to pop up on the console. You will see the details of the requests that were blocked by WAF.

![perimeter_create_8](/assets/images/waf/perimeter_create_8.png)

9.- We suppose now that you have fixed the vulnerability, and you don't need the virtual patching any more. Go to CloudFront console and remove the AWS WAF Web ACL association.

![perimeter_create_9](/assets/images/waf/perimeter_create_9.png)


## Automation honeypot

In this section, you will implement a honeypot that catches and block bad bots using AWS WAF automation capabilities and other AWS services. In fact, you will create a honeypot endpoint using API Gateway and Lambda. This endpoint will be banned using Robot.txt. The _robots.txt_ file is a simple text file placed on your origin which tells webcrawlers like Googlebot if they should access a file or not. In general, bad bots will not honor robot.txt, and scrape the endpoint. When it happens, a Lambda function is triggered and used to block the bad bot IP in AWS WAF automatically.

1.- Deploy the CloudFormation template referenced in the link 'Launch Solution for CloudFront' in this page
[https://docs.aws.amazon.com/solutions/latest/aws-waf-security-automations/deployment.html#step1](https://docs.aws.amazon.com/solutions/latest/aws-waf-security-automations/deployment.html#step1). This template contains multiple automation capabilities, but you will only select the honey pot deployment (Active Bad Bot Protection).
 
![perimeter_honey_1a](/assets/images/waf/perimeter_honeypot_1a.png)

<br/>

![perimeter_honey_1b](/assets/images/waf/perimeter_honeypot_1b.png)

2.- The deployment takes a couple of minutes to finish. When the status of the stack is _CREAT_COMPLETE_, check the output tab and note the URL of the honey pot as well as the name of the WAF Web ACL.

![perimeter_honey_2a](/assets/images/waf/perimeter_honeypot_2a.png)

<br/>

![perimeter_honey_2b](/assets/images/waf/perimeter_honeypot_2b.png)

3.- Go to CloudFront console and create a new origin in your previously created distribution. Use the honeypoint URL noted in the previous step. Make sure to select HTTPS as origin Protocol policy.

![perimeter_honey_3](/assets/images/waf/perimeter_honeypot_3.png)

4.- Next create a new behaviour using this origin, and with _/honeypot_ path pattern. Make sure to Customize Object Caching, and use Zero value in all TTLs.

![perimeter_honey_4](/assets/images/waf/perimeter_honeypot_4.png)

5.- Associate your distribution to the WAF Web ACL created by the CloudFormation template.

![perimeter_honey_5](/assets/images/waf/perimeter_honeypot_5.png)

6.- Normally, you need to modify the _robots.txt_ file in the root of your website to explicitly disallow the honeypot link, as follows as described in solution documentation. For the sake of simplicity, you will skip this part.

7.- Test your api URL, it should be working normally.

![perimeter_honey_7](/assets/images/waf/perimeter_honeypot_7.png)

8.- Using your browser, trigger the honeypot using URL: ```https://[your_cloudfront_distribution_domain]/honeypot```. You will get the below message with your own IP.

![perimeter_honey_8](/assets/images/waf/perimeter_honeypot_8.png)

9.- Go to AWS _WAF_ console, and check that your IP is automatically added in the BAD Bot set.

![perimeter_honey_9](/assets/images/waf/perimeter_honeypot_9.png)

10.- Try to load your api URL again, you will be blocked and you will see the custom error page by CloudFront.

![perimeter_honey_10](/assets/images/waf/perimeter_honeypot_10.png)


## Conclusion

In this Lab, you have learned how to mitigate quickly a vulnerability in your application code with AWS WAF, and how to use AWS WAF to automatically block bad behaving bots along with other AWS services like Amazon API Gateway and AWS Lambda. We highly recommend you to visit the Marketplace for AWS WAF Seller Manager rules in the WAS WAF console. You will see the different WAF rules that are provided by Security Seller to mitigate against bad bot, protect against OWASP TOP 10 vulnerabilities and virtual patching for popular CMS and web servers.