## Configure Origin Group

In this section, you will configure an origin group to provide rerouting during a failover event. You can associate an origin group with a cache behavior to have requests routed from a primary origin to a secondary origin for failover.

1.- In _S3_ console, create a new S3 bucket in another region **US West (N. California)** _us-west-1_ and 
![accelerate_origingroup_1a](/assets/images/cloudfront/accelerate_origingroup_1a.png) 

<br/>

![accelerate_origingroup_1b](/assets/images/cloudfront/accelerate_origingroup_1b.png)  

For _Bucket settings for Block Public Access_, uncheck **Block all public access**. Validate the configurations, check **I acknowledge...** and click **Create bucket**.

![accelerate_origingroup_1c](/assets/images/cloudfront/accelerate_origingroup_1c.png) 

<br/>

Set static website hosting for this. This bucket will serve as a secondary and custom origin.

![accelerate_origingroup_1d](/assets/images/cloudfront/accelerate_origingroup_1d.png)  

<br/>

![accelerate_origingroup_1e](/assets/images/cloudfront/accelerate_origingroup_1e.png) 

<br/>

2.- Create _new-index.html_ file on your computer with the below HTML content (or <a href="/assets/files/accelerate/new-index.html" download>click here to download</a>.); and upload it to your new S3 bucket in **US West (N. California)** _us-west-1_ from the _S3_ console. Make the _new-index.html_ public read.

```html
<html lang="en">
  <body>
    <h1>CloudFront Lab</h1>
    Hi, this is a page from my secondary Origin! We now support Origin group and failover!
  </body>
</html>
```
![accelerate_origingroup_2a](/assets/images/cloudfront/accelerate_origingroup_2a.png) 

<br/>

![accelerate_origingroup_2b](/assets/images/cloudfront/accelerate_origingroup_2b.png) 

<br/>
 
3.- Create a new Origin with the newly created S3 bucket in **US West (N. California)** _us-west-1_.

*	Origin Domain Name : The website hosting Endpoint of Your New S3 bucket (like http://cloudfrontlab-s3bucket-secondary.s3-website-us-west-1.amazonaws.com)

![accelerate_origingroup_3a](/assets/images/cloudfront/accelerate_origingroup_3a.png) 

<br/>

![accelerate_origingroup_3b](/assets/images/cloudfront/accelerate_origingroup_3b.png) 

Review the configuration, then click on **Create** button.

<br/>

4.- Create an _Origin Group_ with a primary and secondary origin. Go to CloudFront _Origins and Origin Groups_ Tab, click **Create Origin Group**. 
 
![accelerate_origingroup_4a](/assets/images/cloudfront/accelerate_origingroup_4a.png) 

Use your _S3-cloudfrontlab-s3bucket_ as primary origin, and your _S3-cloudfrontlab-s3bucket-secondary_ as secondary origin.
For failover criteria, choose **404 Not Found** and **403 Forbidden**.
 
![accelerate_origingroup_4b](/assets/images/cloudfront/accelerate_origingroup_4b.png) 

<br/>

5.- Change default behavior to use Origin Group, so we can test failover. Go to CloudFront _Behaviors_ Tab, select Default(\*), and click **Edit**. 

![accelerate_origingroup_5a](/assets/images/cloudfront/accelerate_origingroup_5a.png) 

Edit the behavior _Origin or Origin Group_ setting to use the _Origin Group_ we created in previous step.
 
![accelerate_origingroup_5b](/assets/images/cloudfront/accelerate_origingroup_5b.png) 

Review your options, then click **Yes, Edit**.

<br/>

6.- After the distribution status changed to _Deployed_, request _new-index.html_ page from CloudFront, you can see your secondary S3 bucket origin serve your request correctly.
 
![accelerate_origingroup_6](/assets/images/cloudfront/accelerate_origingroup_6.png) 
 
<br/>

## Conclusion

!!! success 
    Congrats! you have completed the Bonus Lab of CloudFront module.  You have learned how to set up an Origin Group to provide rerouting during a failover event.