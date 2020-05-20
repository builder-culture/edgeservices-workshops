## Create a new Cache Behavior

In this section you will create a new cache behavior in the CloudFront distribution that you created in the [_Accelerate your content_](/CloudFront/Accelerate_your_content/) lab. You will use this cache behavior to associate the Lambda@Edge function that you will be writing.

1.- Go to the _Amazon CloudFront_ console and select the distribution you created in the [_Accelerate your content_](/CloudFront/Accelerate_your_content/) lab. Go to the cache _Behaviors_ tab and click **Create Behavior**.

![generate_cache_1](/assets/images/lambda-at-edge/generate_cache_1.png)

<br/>

2.- Configure your new cache behavior with the following settings and leave the remaining to settings to the default values :

*	Path : ```/serverless```
*	Origin : _<Select the **EC2 origin** created in Accelerate your content lab\>_
*	Cache Based on Selected Request Headers : **All**

And then click on **Create** button.

![generate_cache_2](/assets/images/lambda-at-edge/generate_cache_2.png)

<br/>

## Create a Lambda@Edge function

1.- Go to the _AWS Console_ and make sure you are in the **US East (N. Virginia)** _us-east-1_ region.

![generate_function_1](/assets/images/lambda-at-edge/generate_function_1.png)

<br/>

2.- Go to the _Lambda_ console and select **Create function**. 

![generate_function_2a](/assets/images/lambda-at-edge/generate_function_2a.png)

<br/>

![generate_function_2b](/assets/images/lambda-at-edge/generate_function_2b.png)

<br/>

Then select **Author from scratch** and configure the following 

*	Name : ```LambdaEdgeImmersionDayLabFunction```
*	Runtime : **Node.js 12.x**

![generate_function_2c](/assets/images/lambda-at-edge/generate_function_2c.png)

<br/>

Expand _Choose or create and execution role_ an configure:

*	Role : **Create new role from AWS policy templates**
*	Role name : `lambda_edge_execution_role`
*	Policy templates: **Basic Lambda@Edge permissions (for CloudFront trigger)**
 
Then click on **Create function** button.

![generate_function_2d](/assets/images/lambda-at-edge/generate_function_2d.png)

You have now created a new IAM role that will be used to allow CloudFront to invoke lambda and write logs to CloudWatch.

<br/>

3.- You'll create a new test event that can be used to test your function locally from the _Lambda_ console. 

* Select from the drop down _Select a test event_ the option **Configure test events**.
 
![generate_function_3](/assets/images/lambda-at-edge/generate_function_3.png)

<br/>

4.- Configure the following:

*	select :  **Create new test event**
*	Event Name : ```TestEvent```
*	Replace the Hello World JSON with the following:

``` json
{
  "Records": [
    {
      "cf": {
        "config": {
          "distributionDomainName": "d123.cloudfront.net",
          "distributionId": "EDFDVBD6EXAMPLE",
          "eventType": "viewer-request",
          "requestId": "MRVMF7KydIvxMWfJIglgwHQwZsbG2IhRJ07sn9AkKUFSHS9EXAMPLE=="
        },
        "request": {
          "clientIp": "2001:0db8:85a3:0:0:8a2e:0370:7334",
          "querystring": "size=large",
          "uri": "/picture.jpg",
          "method": "GET",
          "headers": {
            "host": [
              {
                "key": "Host",
                "value": "d111111abcdef8.cloudfront.net"
              }
            ],
            "user-agent": [
              {
                "key": "User-Agent",
                "value": "curl/7.51.0"
              }
            ]
          },
          "origin": {
            "custom": {
              "customHeaders": {
                "my-origin-custom-header": [
                  {
                    "key": "My-Origin-Custom-Header",
                    "value": "Test"
                  }
                ]
              },
              "domainName": "example.com",
              "keepaliveTimeout": 5,
              "path": "/custom_path",
              "port": 443,
              "protocol": "https",
              "readTimeout": 5,
              "sslProtocols": [
                "TLSv1",
                "TLSv1.1"
              ]
            },
            "s3": {
              "authMethod": "origin-access-identity",
              "customHeaders": {
                "my-origin-custom-header": [
                  {
                    "key": "My-Origin-Custom-Header",
                    "value": "Test"
                  }
                ]
              },
              "domainName": "my-bucket.s3.amazonaws.com",
              "path": "/s3_path",
              "region": "us-east-1"
            }
          }
        }
      }
    }
  ]
}
```

<br/>

5.- Review an click on **Create** button.

![generate_function_5](/assets/images/lambda-at-edge/generate_function_5.png)

<br/>

6.- Now lets write a function to generate the html produced in the [_Accelerate your content_](/CloudFront/Accelerate_your_content/) lab. Copy and paste the code below into the function code window. 

``` javascript
exports.handler= (event, context, callback) => {

    console.log("Request Event:" + JSON.stringify(event, null, 2));
  
    const requestHeaders = event.Records[0].cf.request.headers;
    
    var htmlContent;
    
    //Insert code to generate the html content content here.
    
    const response = {
        status: '200',
        statusDescription: 'OK',
        headers: {
            'cache-control': [{
                key: 'Cache-Control',
                value: 'max-age=100'
            }],
            'content-type': [{
                key: 'Content-Type',
                value: 'text/html'
            }],
            'content-encoding': [{
                key: 'Content-Encoding',
                value: 'UTF-8'
            }],
        },
        body: htmlContent,
    };
        
    callback(null, response);
}
```
<br/>

Click on **Save** button.

![generate_function_6](/assets/images/lambda-at-edge/generate_function_6.png)

Then test the function using the TestEvent configured.

<br/>

7.- Look at the **Execution Result**. 

![generate_function_7a](/assets/images/lambda-at-edge/generate_function_7a.png)

<br/>

Notice that the output is a JSON representation of the HTTP 200 response that CloudFront will use to respond to the request. In this case, the response is still missing the body.  In the Log output section, notice that the test event that we configured in step 6 logged as the Request Event on the input of the function.  This JSON represents attributes of the request received by CloudFront which can be read or modified. In this exercise we will read the headers and return the results in a pretty HTML table.

![generate_function_7b](/assets/images/lambda-at-edge/generate_function_7b.png)

<br/>

8.- Replace the comment with the code needed to generate the html body. You can use console.log to output to troubleshoot your code. The complete solution is the following:

``` javascript
exports.handler= (event, context, callback) => {

    const requestHeaders = event.Records[0].cf.request.headers;
   
    var str = `<table border="1" width="100%">
                    <thead>
                        <tr><td><h1>Header</h2></td><td><h1>Value</h2></td></tr>
                    </thead>
                    <tbody>`;
                    
    for (var key in requestHeaders) {
      if (requestHeaders.hasOwnProperty(key)) {
        str += "<tr><td>"+key + "</td><td>" + requestHeaders[key][0].value + "</td></tr>";
      }
    }
    
    str+= "</tbody></table>";
   
    var htmlContent = `<html lang="en">
                  <body>
                    <table border="1" width="100%">
                    <thead>
                        <tr><td><h1>Lambda@Edge Lab</h1></td></tr>
                    </thead>
                    <tfoot>
                        <tr><td>Immersion Days - Edge Services - Module 3</td></tr>
                    </tfoot>
                    <tbody>
                        <tr><td>Response sent by API</td></tr>
                    </tbody>
                    <tbody>
                        <tr><td> ` + str + `</td></tr>
                    </tbody>
                    </table>
                  </body>
                </html>`;
    

    const response = {
        status: '200',
        statusDescription: 'OK',
        headers: {
            'cache-control': [{
                key: 'Cache-Control',
                value: 'max-age=100'
            }],
            'content-type': [{
                key: 'Content-Type',
                value: 'text/html'
            }],
            'content-encoding': [{
                key: 'Content-Encoding',
                value: 'UTF-8'
            }],
        },
        body: htmlContent,
    };
        
    callback(null, response);
}
```

<br/>

9.- Once you've completed testing of your function. Publish and deploy the first version of the function. 

Select the _Actions_ drop down and select **Publish new version**.  

![generate_function_9](/assets/images/lambda-at-edge/generate_function_9.png)

<br/>

10.- Specify a version description then click **Publish**.

![generate_function_10](/assets/images/lambda-at-edge/generate_function_10.png)

Congratulations you now have a Lambda Function that can be used with CloudFront!

<br/>

## Deploy Lambda@Edge function to CloudFront

In this section we will add CloudFront as a trigger for you Lambda Function.

1.- In the same lambda console, you now have Version 1 of the function deployed. 

Click on **Add trigger** button.

![generate_deploy_1a](/assets/images/lambda-at-edge/generate_deploy_1a.png)

<br/>

And select **CloudFront** option.

![generate_deploy_1b](/assets/images/lambda-at-edge/generate_deploy_1b.png)

<br/>

2.- Click on **Deploy to Lambda@Edge** button.
 
![generate_deploy_2](/assets/images/lambda-at-edge/generate_deploy_2.png)

<br/>

3.- Configure the trigger to use the CloudFront Distribution and Cache Behavior created earlier with the following setting,   

*	Distribution : **<select the distributionID created from _Accelerate your content_ lab\>**
*	Cache behavior : ```/serverless```
*	CloudFront event :  **Origin request**
*	Selected _I acknowledge that on deploy a new version of this function will be published with the above trigger and replicated across all available AWS regions._

Click on **Deploy** button.

![generate_deploy_3](/assets/images/lambda-at-edge/generate_deploy_3.png)

It will take approximately 15 minutes for a full deployment to your CloudFront distribution. In some cases, you maybe able to begin testing it within 5 minutes depending on your location.

<br/>

4.- To view the replica functions to return to the Lambda main console view to list the your functions. Then select the **gear icon** on the top right to configure your preferences.

![generate_deploy_4](/assets/images/lambda-at-edge/generate_deploy_4.png)

<br/>

5.- Select the check box for **Show replica functions** and click **Save**.

![generate_deploy_5](/assets/images/lambda-at-edge/generate_deploy_5.png)

<br/>

6.- Now search for your function in the function list and you will find a Replica function in us-east-1 for the function you created. When you switch to other AWS regions, you will find that there is a replica function in all of the regions where CloudFront has a Regional Edge Cache. These are the functions that will be invoked when your CloudFront distribution executes Lambda@Edge.

![generate_deploy_6](/assets/images/lambda-at-edge/generate_deploy_6.png)

<br/>

7.- Once your cloudfront distribution has completed deploying, test your CloudFront distribution by going to the Distribution on the browser with the serverless path: [i.e. https://abcdefghxyz.cloudfront.net/serverless]. 

![generate_deploy_7](/assets/images/lambda-at-edge/generate_deploy_7.png)

Congratulations! You have successfully deployed your lambda@edge function to dynamically generate a response.

<br/>

## Metrics and Logs

In this section, you will learn where to view Lambda@Edge metrics and CloudWatch logs for your function.

1.- To generate some traffic from different geographies, use a free website such as [https://www.geoscreenshot.com](https://www.geoscreenshot.com) to send a request to your distribution to hit the serverless path (i.e. https://d123.cloudfront.net/serverless) from different locations.  Submit the request several times to generate traffic to CloudFront from different regions.

<br/>

2.- Go to the AWS Lambda console and select an AWS region near one of the geographies that you selected to submit the request from from step 1.

<br/>

3.- Ensure that you have your preferences configured to _show replica functions_ as configured in the previous sections. Then select the replica function for that region and view the _Monitoring_ tab for that function.

![generate_metrics_3](/assets/images/lambda-at-edge/generate_metrics_3.png)

<br/>

4.- To view logs from your function execution, select _View logs in CloudWatch_.  Here you can view any logs that are generated from your function invocations. Note, any log lines generated from your code will also appear here.

![generate_metrics_4](/assets/images/lambda-at-edge/generate_metrics_4.png)

!!! Tip
    Metrics and Logs are not aggregated globally. Switch between different AWS regions to view metrics and logs for function invocations in each regions.

<br/>

## Conclusion

!!! success
    Congratulations! You have completed the lab and now have an understanding of how to configure and deploy Lambda@Edge.