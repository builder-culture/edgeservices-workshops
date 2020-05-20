## Intro to a vulnerability

### OWASP Top 10 vulnerability mitigation using AWS WAF

In this section, you will use AWS WAF to mitigate a vulnerability caused by weak code on your server. The vulnerability is a Local File Inclusion (LFI) in the Broken Access Control category of OWASP Top 10 (A5 category in 2017 release). Let's suppose that this vulnerability was introduced recently by code a change, where your developer has added to your application the capability to show additional information about itself using info query string. 

<br/>

### Test the api vulnerability in your browser

* `https://[your_distribution_domain_name]/api` : will give the same results as before. Nothing new here.

* `https://[your_distribution_domain_name]/api?info=about` : will append to the response, the content of `about` file.

![perimeter_create_1a](/assets/images/waf/perimeter_create_0a.png)

* `https://[your_distribution_domain_name]/api?info=dependency;cat%20/etc/passwd` : will append to the response, the content of `dependency` file and `/etc/passwd`. 

![perimeter_create_1b](/assets/images/waf/perimeter_create_0b.png)

<br/>

## Create a virtual patch (vulneravility mitigation) 

1.- While waiting for your developers to correct the code and enforce access control, you will mitigate the vulnerability using AWS WAF. Lets go.

Go to AWS _WAF_ Console

![perimeter_create_1a](/assets/images/waf/perimeter_create_1a.png)

<br/>

 From the menu on the left select the **Stwich to AWS WAF Classic** option.

![perimeter_create_1b](/assets/images/waf/perimeter_create_1b.png)

<br/>


2.- Create a new condition. From the menu on the left, in _Conditions_ section select **String and regex matching**.

![perimeter_create_2a](/assets/images/waf/perimeter_create_2a.png)

<br/>

Click on  the **Create condition** button.

![perimeter_create_2b](/assets/images/waf/perimeter_create_2b.png)

<br/>

Create a new String match condition with the below filter settings.

On _Create string match condition_ section :

* Name : `Remote file inclusion condition` 
* Region : **Globlal (CloudFront)**
* Type : **String match**

On _Filter settings_ section:

* Part of the request to filter on : **Single query parameter (value only)**
* Query parameter name : `info`
* Match type : **Contains**
* Transformation : **URL Encode** 
* Value is base64-encoded : **unchecked**
* Value to match : `/`

Review the configuration and click on **Add filter** button.

![perimeter_create_2c](/assets/images/waf/perimeter_create_2c.png)

<br/>


3.- Create another filter in the same condition with the following filter.

On _Create string match condition_ section :

* Name : `Remote file inclusion condition` 
* Region : **Globlal (CloudFront)**
* Type : **String match**

On _Filter settings_ section:

* Part of the request to filter on : **Single query parameter (value only)**
* Query parameter name : `info`
* Match type : **Contains**
* Transformation : **URL Encode** 
* Value is base64-encoded : **unchecked**
* Value to match : `;`
 
Review the configuration and click on **Add filter** button.

![perimeter_create_3](/assets/images/waf/perimeter_create_3.png)

<br/>

4.- To finalize and create a condition that detects `"/"` and `";"` characters in the info query string after decoding as URL, click on the **Create** button.

![perimeter_create_4](/assets/images/waf/perimeter_create_4.png)

<br/>

5.- Create a new rule in AWS _WAF_ console. 

From the menu on the left, select the **Rules** option.

![perimeter_create_5a](/assets/images/waf/perimeter_create_5a.png)

<br/>

Then click on **Create rule** button.

![perimeter_create_5b](/assets/images/waf/perimeter_create_5b.png)

<br/>

Create a rule with the bellow settings, first On _Create rule_ section:

* Name : `LFI protection`
* CloudWatch metric name : `LFIprotection`
* Rule type : **Regular rule**
* Region **Global (CloudFront)**

Then, On _Add conditions_ section select the following options:

**When a request**: 

* _does_
* _match at least one of the filters in the string match condition_
* _Remote file inclusion condition_ 

After doing the previous selection, automatically it shows the prevouly created _string matching condition_ details.

Review your configuratin anc click on the **Create** button.

![perimeter_create_5c](/assets/images/waf/perimeter_create_5c.png)

<br/>

6.- Finally, create a new WebACL and associate it with your CloudFront distribution. In this WebACL, you will permit all requests unless they match the rule you have just created. Confirm, review and proceed with WebAcl Creation.

From the menu on the left, select **Web ACLs** option.

![perimeter_create_6a](/assets/images/waf/perimeter_create_6a.png)

<br/>

Click on the **Create web ACL** button.

![perimeter_create_6b](/assets/images/waf/perimeter_create_6b.png)

<br/>

Setup a _Web ACL with_ the following options:

* Web ACL name : `app virtual patching`
* CloudWatch metric name : `appvirtualpatching`
* Region : **Global (CloudFront)**
* AWS resource to associate `<Select_Your_CloudFront_Distribution>`

Review and click **Next**.

![perimeter_create_6c](/assets/images/waf/perimeter_create_6c.png)

<br/>

Click **Next** in _Create conditions_ 

![perimeter_create_6d](/assets/images/waf/perimeter_create_6d.png)

<br/>

On _Add rules to a Web ACL_ section: 

 *  Rules : **LFI protection**

Then click on the **Add another Rule** button.

![perimeter_create_6e](/assets/images/waf/perimeter_create_6e.png)

<br/>
Select the following options:

* On the _FI protection_ Rule : **Block**
* On _Default Action_ : **Allow all requests that don't match any rules**

Click on the **Review and create** button.

![perimeter_create_6f](/assets/images/waf/perimeter_create_6f.png)

<br/>

Review and click on the **Confirm and create** button.

![perimeter_create_6g](/assets/images/waf/perimeter_create_6g.png)

<br/>

7.- In a few seconds, the change will propgate to AWS WAF. Test again in your browser the attacker URL: `https://[your_distribution_domain_name]/api?info=dependency;cat%20/etc/passwd`. You will be blocked (403 Forbidden), and the custom error page will be returned by CloudFront.

![perimeter_create_7](/assets/images/waf/perimeter_create_7.png)

<br/>

8.- Finally, go to your newly created Web ACL in the AWS WAF console and check the sampled requests. It takes a couple of minutes to pop up on the console. 

Select the recently created Web ACL.

![perimeter_create_8a](/assets/images/waf/perimeter_create_8a.png)

<br/>

On _Sampled resuests_ section select **LFI protection**; and click on the **Get new samples** button.

![perimeter_create_8b](/assets/images/waf/perimeter_create_8b.png)

<br/>

You will see the details of the requests that were blocked by WAF.

![perimeter_create_8c](/assets/images/waf/perimeter_create_8c.png)

<br/>

9.- We suppose now that you have fixed the vulnerability, and you don't need the virtual patching any more. Go to CloudFront console and remove the AWS WAF Web ACL association.

Click on the **Edit** button.

![perimeter_create_9a](/assets/images/waf/perimeter_create_9a.png)

<br/>

 Set _AWS WAF Web ACL_ to **None**.

![perimeter_create_9b](/assets/images/waf/perimeter_create_9b.png)

<br/>

Click on the **Yes, Edit** button.

![perimeter_create_9c](/assets/images/waf/perimeter_create_9c.png)

<br/>


## Honeypot Automation

In this section, you will implement a honeypot that catches and block bad bots using AWS WAF automation capabilities and other AWS services. In fact, you will create a honeypot endpoint using API Gateway and Lambda. This endpoint will be banned using _robots.txt_. The _robots.txt_ file is a simple text file placed on your origin which tells webcrawlers like Googlebot if they should access a file or not. In general, bad bots will not honor robot.txt, and scrape the endpoint. When it happens, a Lambda function is triggered and used to block the bad bot IP in AWS WAF automatically.

1.- Deploy the _CloudFormation_ template referenced in the link **Launch Solution ** for _CloudFront_ in this [page](https://docs.aws.amazon.com/solutions/latest/aws-waf-security-automations/deployment.html#step1).

[Launch Solution](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=AWSWAFSecurityAutomations&templateURL=https:%2F%2Fs3.amazonaws.com%2Fsolutions-reference%2Faws-waf-security-automations%2Flatest%2Faws-waf-security-automations.template) 

 
![perimeter_honey_1a](/assets/images/waf/perimeter_honeypot_1a.png)

<br/>

This template contains multiple automation capabilities, but you will only select the honey pot deployment (**Active Bad Bot Protection**).

![perimeter_honey_1b](/assets/images/waf/perimeter_honeypot_1b.png)

<br/>

Click on the **Next** button.

![perimeter_honey_1c](/assets/images/waf/perimeter_honeypot_1c.png)

<br/>

* Check the two check box of **I aknowledge that..."**

Click on the **Create stack** button.

![perimeter_honey_1d](/assets/images/waf/perimeter_honeypot_1d.png)

<br/>

2.- The deployment takes a couple of minutes to finish. When the status of the stack is _CREATE_COMPLETE_, check the output tab and note the URL of the honey pot as well as the name of the WAF Web ACL.

![perimeter_honey_2a](/assets/images/waf/perimeter_honeypot_2a.png)

<br/>

!!! note 
    On _Outputs_ tab note the _BadBotHoneypotEndpoint_ value.

    **Yo will need this value in next steps**

![perimeter_honey_2b](/assets/images/waf/perimeter_honeypot_2b.png)

<br/>

3.- Go to _CloudFront console_ to create a new origin in your previously created distribution. 

Go to _Origins and Origin Group_ Tab and click on the **Create Origin** button. 

![perimeter_honey_3a](/assets/images/waf/perimeter_honeypot_3a.png)

<br/>

Use the honeypoint URL noted in the previous step. Make sure to select HTTPS as origin Protocol policy. Use the following settings and 

* Origin Domain Name : <Generated_BadBotHoneypotEndpoint>
* Origin Path : `/ProdStage`

* Minimum Origin SSL Protocol : **TLSv1.2**
* Origin Protocol Policy : **HTTPS Only**

Click on **Create**.

![perimeter_honey_3b](/assets/images/waf/perimeter_honeypot_3b.png)

<br/>

4.- Go to _Behaviors_ Tab. Click on the **Create Behavior** button.

![perimeter_honey_4a](/assets/images/waf/perimeter_honeypot_4a.png)

<br/>

Next create a new behaviour using the recently created origin, and with `/honeypot` path pattern. Make sure to Customize Object Caching, and use Zero value in all TTLs.

* Path Pattern : `/honeypot`
* Origin or Origin Group : `<Your_custom_generated_BadBotHoneypotEndpoint_/ProdStage>`
* Viewer Protocol Policy **Redirect HTTP to HTTPS**

* Object Caching : **Customize**

    * Minimum TTL : `0`
    * Maximum TTL : `0`
    * Default TTL : `0`

![perimeter_honey_4b](/assets/images/waf/perimeter_honeypot_4b.png)

<br/>
Click on **Create**

![perimeter_honey_4c](/assets/images/waf/perimeter_honeypot_4c.png)

<br/>

5.- Associate your distribution to the WAF Web ACL created by the CloudFormation template.

Go to _CloudFront_ Console; and click on **Edit**.

![perimeter_honey_5a](/assets/images/waf/perimeter_honeypot_5a.png)

<br/>

Select the  WAF Web ACL created by the CloudFormation template (i.e. `AWSWAFSecurityAutomations (wafv1)`).

![perimeter_honey_5b](/assets/images/waf/perimeter_honeypot_5b.png)

<br/>

Review and click on the **Yes, Edit** button.

![perimeter_honey_5c](/assets/images/waf/perimeter_honeypot_5c.png)

<br/>

6.- Normally, you need to modify the _robots.txt_ file in the root of your website to explicitly disallow the honeypot link, as follows as described in solution documentation. For the sake of simplicity, you will skip this part.

<br/>

7.- Test your api URL, it should be working normally.

![perimeter_honey_7](/assets/images/waf/perimeter_honeypot_7.png)

<br/>

8.- Using your browser, trigger the honeypot using URL: `https://[your_distribution_domain_name]/honeypot`. You will get the below message with your own IP.

![perimeter_honey_8](/assets/images/waf/perimeter_honeypot_8.png)

<br/>

9.- Go to AWS _WAF_ console, and check that your IP is automatically added in the BAD Bot set.

On the AWS _WAF_ console, select from the left menu the **Web ACLs** option. And select the recently created WebACL (i.e. `AWSWAFSecurityAutomations`)

![perimeter_honey_9a](/assets/images/waf/perimeter_honeypot_9a.png)

<br/>

Go to **Rules** Tab and pick one with Action equals to _Block request_.

![perimeter_honey_9b](/assets/images/waf/perimeter_honeypot_9b.png)

<br/>

Check that your IP is automatically added 

![perimeter_honey_9c](/assets/images/waf/perimeter_honeypot_9c.png)

<br/>

10.- Try to load your api URL again (i.e. `https://[your_distribution_domain_name]/api?info=about`), you will be blocked and you will see the custom error page by CloudFront.

![perimeter_honey_10](/assets/images/waf/perimeter_honeypot_10.png)

<br/>

## Conclusion

!!! success
    Congratulations! In this Lab, you have learned how to mitigate quickly a vulnerability in your application code with AWS WAF, and how to use AWS WAF to automatically block bad behaving bots along with other AWS services like Amazon API Gateway and AWS Lambda. We highly recommend you to visit the Marketplace for AWS WAF Seller Manager rules in the WAS WAF console. You will see the different WAF rules that are provided by Security Seller to mitigate against bad bot, protect against OWASP TOP 10 vulnerabilities and virtual patching for popular CMS and web servers.