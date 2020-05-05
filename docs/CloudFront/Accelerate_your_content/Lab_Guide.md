## Create EC2 & S3 origins

In this section, you will create both S3 and EC2 origin using a provided CloudFormation template.

1.- Go to CloudFormation console in North Virginia _us-east-1_ and create a new stack

![accelerate_origin_1](/assets/images/cloudfront/accelerate_origin_1.png)

2.- Choose _**Template is ready**_ and _**Upload a template file**_, then choose and upload the following template file:
  cloudfront-lab.yaml
  
  >  link to resources.

![accelerate_origin_2](/assets/images/cloudfront/accelerate_origin_2.png)

3.- Click Next, enter a name for your stack and click on Next twice (Options and Review) , then click Create stack.

![accelerate_origin_3](/assets/images/cloudfront/accelerate_origin_3.png)

4.- Finally, when the launch is completed, go to the Outputs Tab and note the domain name of your EC2 based web server as well as the S3 bucket name. During stack launch process, you can check progress via Events Tab and get more details for debugging in case of any problem.

![accelerate_origin_4a](/assets/images/cloudfront/accelerate_origin_4a.png)

<br/>

![accelerate_origin_4b](/assets/images/cloudfront/accelerate_origin_4b.png)

5.- Create index.html file on your computer with the below HTML content. This HTML calls the dynamic content using an iframe tag. In fact, when a user makes a request  for index.html, the browser sends a subsequent request to /api.

```html  
<!DOCTYPE html>
<html lang="en">
  <body>
    <table border="1" width="100%">
    <thead>
        <tr><td><h1>CloudFront Lab</h1></td></tr>
    </thead>
    <tfoot>
        <tr><td>Immersion Days - Edge Services - Module 1</td></tr>
    </tfoot>
    <tbody>
        <tr><td>Response sent by API</td></tr>
    </tbody>
    <tbody>
        <tr><td> 
          <iframe src='/api' style="width:100%; height:100%;"></iframe>
        </td></tr>
    </tbody>
    </table>
  </body>
</html>
```

6.-  Upload index.html file to the S3 bucket created by CloudFormation and leave all settings to default.

![accelerate_origin_6](/assets/images/cloudfront/accelerate_origin_6.png)

7.-  When you try to request index.html using the S3 provided url, the access will be denied since it's not configured as public object.

![accelerate_origin_7a](/assets/images/cloudfront/accelerate_origin_7a.png)

<br/>

![accelerate_origin_7b](/assets/images/cloudfront/accelerate_origin_7b.png)

8.- The CloudFormation template has deployed a Node.Js based application that listens to HTTP requests on port 80 of the EC2 instance. Upon receiving a request, the application will send back a json response that includes the headers received in the request.  It will also inspect the query string info and return some data from the webserver based on the query string value. The application code is simply the below:

```javascript
const express = require('express')
const app = express()

app.get('/api', function (req, res) {
  console.log(JSON.stringify(req.headers))
  if (req.query.info) {
    require('child_process').exec('cat '+ req.query.info,
      function (err, data) {
        res.send(new Date().toISOString() + '\n' + JSON.stringify(req.headers)+ '\n'+data)
      });
  } else {
    res.send(new Date().toISOString() + '\n' + JSON.stringify(req.headers))
  }
});

app.listen(8080, function () {
  console.log('api is up!')
})
```

Make sure the application work by going requesting http://[instance-domain-name]/api.  You should see the below screen in your browser.

![accelerate_origin_8](/assets/images/cloudfront/accelerate_origin_8.png)

## Create CloudFront Distribution

1.-  Go to CloudFront console and create a new distribution. If this is your first time to use CloudFront, click on Create Distribution in the Getting Started page. Otherwise, you can get started with a Web distribution.
 
![accelerate_distribution_1](/assets/images/cloudfront/accelerate_distribution_1.png) 

2.- Configure the default origin to the previously created S3 bucket and grant CloudFront the permissions to the bucket using Origin Access Identity settings:

*	Origin Domain Name: ```Your S3 bucket```
*	Restrict Bucket Access: ```Yes```
*	Origin Access Identity: ```Create a new Identity```
*	Grant Permissions on Bucket: ```Yes, Update Bucket policy```
 
![accelerate_distribution_2](/assets/images/cloudfront/accelerate_distribution_2.png) 


3.- Configure the default behaviour to below

*	Viewer Protocol Policy: ```Redirect HTTP to HTTPS```
*	Object Caching: ```Customize with Minimum TTL to 86400```
*	Compress Objects Automatically: ```Yes```
 
![accelerate_distribution_3](/assets/images/cloudfront/accelerate_distribution_3.png) 

4.- Finally, configure root object to index.html and leave the rest to defaults. In this lab, you will be using a domain name provided by CloudFront, however, if you want to use your own domain name, you can configure it with Alternate Domain Names (CNAMEs) section. 


Click Create distribution, CloudFront start creating the distribution and normally it takes 10 to 20 mins to fully propagate. The status of the distribution will be In Progress. To check the status, you need to click on the Distribution menu on left pane.

![accelerate_distribution_4a](/assets/images/cloudfront/accelerate_distribution_4a.png) 

<br/>

![accelerate_distribution_4b](/assets/images/cloudfront/accelerate_distribution_4b.png) 

<br/>

![accelerate_distribution_4c](/assets/images/cloudfront/accelerate_distribution_4c.png) 

5.- In the distributions console, click on your distribution ID, then go to Origins and Origin Groups Tab to create another origin for the api. Click Origins and Origin Groups tab then click Create Origin button.
 
![accelerate_distribution_5](/assets/images/cloudfront/accelerate_distribution_5.png) 

6.- Configure the origin using EC2 instance domain name, and increase keep alive timeout to 60 seconds. Please note that although we want to serve content on HTTPS to users, we want to keep HTTP connection the origin to reduce the TLS overhead on the origin. This is configured by setting the  Origin Protocol Policy to HTTP only as per the below screenshot.
 
![accelerate_distribution_6](/assets/images/cloudfront/accelerate_distribution_6.png) 

7.- Create a new behaviour for the api. Select Behavior tab then click Create Behabior button.
 
![accelerate_distribution_7](/assets/images/cloudfront/accelerate_distribution_7.png) 

8.- Configure a second cache behaviour to use the EC2 origin with following parameters to use CloudFront as proxy and bypass any caching layers.

*	Path Pattern: /api
*	Viewer Protocol Policy: Redirect HTTP to HTTPS
*	Cache Based on Selected Headers: All
*	Forward Cookies: All
*	Query String Forwarding and Caching:Forward all, cache based on all.
*	Compress Objects Automatically: Yes

![accelerate_distribution_8](/assets/images/cloudfront/accelerate_distribution_8.png) 

9.- Check the unique domain name that CloudFront has associated to your distribution in the General tab.
 
![accelerate_distribution_9](/assets/images/cloudfront/accelerate_distribution_9.png) 

## Test the application on CloudFront

1.- CloudFront distribution can be ready to be used locally even if the status is still in progress, since the status will change to deployed when the propagation has reached all 176 edge locations. To test if the distribution is ready to be used locally, you can lookup its CloudFront domain name in command line by nslookup command. Keep in mind that dxxxx.cloudfront.net name is unique for every distribution, this is why need to use your own distribution value for testing. Note how CloudFront returns multiple IPs for each DNS query to increase application resiliency.
 
![accelerate_test_1](/assets/images/cloudfront/accelerate_test_1.png) 

2.- When the propagation is done, you can test the webpage on your browser as served by CloudFront using http://dxxxx.cloudfront.net. In the webpage, you can see the different headers that CloudFront has forwarded and appended to your API endpoint:

*	cloudfront-forwarded-proto: Indicates the protocol used by the viewer to connect to CloudFront
*	cloudfront-is-mobile-viewer: Indicates the viewer's device type
*	cloudfront-viewer-country: Indicates the viewer's country
*	x-amz-cf-id: a unique id for this request provided by CloudFront. IF you refresh the webpage, you will see that how the request id is changing. It's useful to log it on your webserver in general. Additionally, this id will be sent back to every viewer request and sent to CloudFront access logs. If you need to debug any issue you can open a support ticket and provide them with the req id. 
Also note how CloudFront redirected the request to HTTPS.

![accelerate_test_2](/assets/images/cloudfront/accelerate_test_2.png) 


3.- If you use the developper tools of your favorite web browser, you can check the response headers sent by CloudFront. Three headers are interesting to check:

*	x-amz-cf-id which holds the request id assigned by CloudFront.
*	x-amz-cf-pop which indicates the CloudFront edge location that served your request. Each edge location is identified by a three-letter code and an arbitrarily assigned number, for example, DFW3. The three-letter code typically corresponds with the International Air Transport Association airport code for an airport near the edge location.
*	x-cache which indicates whether the request was a cache hit or a cache miss. Normally, for your html file, you will get a 'Hit from Cloudfront' value in the subsequent requests, but always 'Miss from CloudFront' for /api request since caching is disabled for this behaviour.
 
![accelerate_test_1](/assets/images/cloudfront/accelerate_test_3.png) 


## Test invalidations

As you saw previously, the main index.html page is in cache and resulting in a Hit from CloudFront. Suppose that you have to change the HTML file but you can't change URL to point to the new version, in this case, you need to invalidate the page. 

1.- Go to the Invalidations Tab

![accelerate_invalidations_1](/assets/images/cloudfront/accelerate_invalidations_1.png) 

2.- Create an invalidation for your index.html. You can specify / in the Object paths because we already set index.html as default root object.

![accelerate_invalidations_2](/assets/images/cloudfront/accelerate_invalidations_2.png)  

3.- After a few of seconds, test again the page using the browser developer tool, and you'll find that it resulted in a cache miss.

![accelerate_invalidations_3](/assets/images/cloudfront/accelerate_invalidations_3.png) 


## Configure custom error page

In this section, you will configure a custom error page for graceful failures if the requested content is not found.

1.- Test a random URL using your CloudFront domain name and you will get a 403 Forbidden response from S3 behind CloudFront because the file does not exist. By default, CloudFront caches this response for 5 minutes.

![accelerate_errorpage_1](/assets/images/cloudfront/accelerate_errorpage_1.png) 

2.- Create error.html file on your computer with the below HTML content and upload it to your S3 bucket in the S3 console.

```html
<html lang="en">
  <body>
    <h1>CloudFront Lab</h1>
    Oups, this is a nice error page!
  </body>
</html>
```

![accelerate_errorpage_2](/assets/images/cloudfront/accelerate_errorpage_2.png) 

3.-  In CloudFront console, go to Error Pages tab and click Create Custom Error Response button.
 
![accelerate_errorpage_3](/assets/images/cloudfront/accelerate_errorpage_3.png) 

4.- Configure custom error response with the following settings.

*	HTTP Error Code: 403 Forbidden
*	Error Cacching Minimum TTL (seconds) : 60
*	Customize Error Response : Yes
*	Response Error Response : /error.html
*	HTTP Response Code: 200 OK
 
![accelerate_errorpage_4](/assets/images/cloudfront/accelerate_errorpage_4.png) 

5.- Test your custom error page, by requesting a random page from CloudFront. You may need a few minutes to wait for distribution to update and propagate to edge locations. Make sure that you use a different random value from the previous test, other wise you will get the same cached version if you test within 5 mins.
 
![accelerate_errorpage_5](/assets/images/cloudfront/accelerate_errorpage_5.png) 

6.- Create another custom error page that will be triggered when the origin is not reachable by CloudFront. Use the following settings:

*	HTTP Error Code: 504 Gateway Timeout
*	Error Caching Minimum TTL (seconds) : 5
*	Customize Error Response : Yes
*	Response Error Response : /error.html
*	HTTP Response Code: 200 OK
 
![accelerate_errorpage_6](/assets/images/cloudfront/accelerate_errorpage_6.png) 

7.- In the EC2 console, stop the EC2 instance which is hosting the Nodejs api.

8.- test again your index.html and wait until the API call fails gracefully to the custom error page.

![accelerate_errorpage_8](/assets/images/cloudfront/accelerate_errorpage_8.png)  

## Configure Origin Group

In this section, you will configure an origin group to provide rerouting during a failover event. You can associate an origin group with a cache behavior to have requests routed from a primary origin to a secondary origin for failover.

1.- In S3 console, create a new S3 bucket in another region us-west-1 and set static website hosting for it. This bucket will serve as a secondary and custom origin.

![origingroup_1](/assets/images/cloudfront/accelerate_origingroup_1a.png) 

<br/>

![origingroup_1](/assets/images/cloudfront/accelerate_origingroup_1b.png)  
 
2.- Create new-index.html file on your computer with the below HTML content and upload it to your new S3 bucket in us-west-1 from the S3 console. Make the new-index.html public read.

```html
<html lang="en">
  <body>
    <h1>CloudFront Lab</h1>
    Hi, this is a page from my secondary Origin! We now support Origin group and failover!
  </body>
</html>
```
 
3.- Create a new Origin with the newly created S3 bucket in us-west-1.
•	Origin Domain Name: The website hosting Endpoint of Your New S3 bucket (like http://cloudfrontlab-s3bucket-secondary.s3-website-us-west-1.amazonaws.com)

![accelerate_origingroup_5a](/assets/images/cloudfront/accelerate_origingroup_3.png) 

4.- Create an Origin Group with a primary and secondary origin. Go to CloudFront Origins and Origin Groups Tab, click Create Origin Group. 
 
![accelerate_origingroup_4a](/assets/images/cloudfront/accelerate_origingroup_4a.png) 

Use S3-cloudfrontlab-s3bucket as primary origin, and S3-cloudfrontlab-s3bucket-secondary as secondary origin.
For failover criteria, choose 404 Not Found and 403 Forbidden.
 
![accelerate_origingroup_4b](/assets/images/cloudfront/accelerate_origingroup_4b.png) 

5.- Change default behavior to use Origin Group, so we can test failover. Go to CloudFront Behaviors Tab, select Default(*), and click Edit. 

![accelerate_origingroup_5a](/assets/images/cloudfront/accelerate_origingroup_5a.png) 

Edit the behavior to use the Origin Group we created in previous step.
 
![accelerate_origingroup_5b](/assets/images/cloudfront/accelerate_origingroup_5b.png) 

6.- After the distribution status changed to Deployed, request new-index.html page from CloudFront, you can see your secondary S3 bucket origin serve your request correctly.
 
![accelerate_origingroup_6](/assets/images/cloudfront/accelerate_origingroup_6.png) 
 

## Conclusion

Congrats! you have completed the Lab of CloudFront module. You have learned how to create a distribution with multiple origins and multiple behaviors that delivers static content and proxies api dynamic content to your origin. You have learned how to set up an Origin Group to provide rerouting during a failover event. You have also learned how to set up custom error pages and invalidate content quickly. Finally, you have learned about some important CloudFront headers for debugging.
