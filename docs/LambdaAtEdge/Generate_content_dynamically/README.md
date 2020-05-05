## Pre-requisite

As a pre-requisite to this lab, you must first complete the “Accelerate your content using CloudFront” lab, as we will be using the distribution you have created there to associate our lambda function.


## Introduction

In this lab,  you will use Lambda@Edge to dynamically generate the page created in Lab 1.  Rather than a simple dump of the request headers, we will output the values in an HTML table.  You will learn how to create Lamdba@Edge permissions, deploy functional Lambda@Edge code, and test it.  Finally, you will be able to view function logs, metrics, and troubleshoot your code.

![generate_intro](/assets/images/lambda-at-edge/generate_intro.png)