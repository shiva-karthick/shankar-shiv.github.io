---
layout: post
title:  "AWS IoT button and IFTTT Webhooks channel"
date:   2020-06-09 11:20:00 +0800
categories: iot
mathjax: true
---

In the past, I have wished to order a pizza in the press of a button. But now, it's very much possible with the assistance of cloud services. The AWS IoT Button seemed a great candidate for many IoT projects. 

There are numerous use cases such as :
+ Call or alert someone
+ Open your garage door
+ Remotely control your home appliances and many more ...

I decided to give it a try. In this guide, I would like to show how I made the AWS IoT Button access the IFTTT's Webhooks Channel and finally create a custom action (ordering a pizza, send SMS to your mobile or whatever :).

Let's get started!

<img src="/assets/images/Iot-button/button.jpg" alt="button" style="width:450;height:500px;">

Things used in this project:
+ Amazon Web Services AWS IoT Button
+ Amazon Web Services AWS Lambda
+ IFTTT Webhooks service
+ Amazon Web Services AWS IoT

## Set up an AWS account
+ Create an <a href="https://aws.amazon.com/console/"> <span style="background-color: black; color : white">AWS console account</span> </a> either as root or with a user that has enough access to configure and manage the IoT devices. .
+ Complete "Getting Started" section: <a href="https://aws.amazon.com/iot/button"> <span style="background-color: black; color : white">https://aws.amazon.com/iot/button</span> </a>
  
  1) In the Console services search bar -> Type "IoT Core" 
    <img src="/assets/images/Iot-button/IoT_core.png" alt="Iot core" style="width:1000px;height:500px;">
  2) In the side bar click on "Manage" and then "Create a single thing" 
    <img src="/assets/images/Iot-button/things.png" alt="things" style="width:1000px;height:500px;">
  3) Leave blank for "Thing Type", "Thing Group".
  
  4) Just follow the steps as directed  :)
   
  5) The last confirmation page should show the “endpoint subdomain” and “endpoint region” information. They tell your button where to send its information when you press it.

## Connect the AWS IoT button to your wifi network

This excellent documentation covers all the necessary information to connect the AWS IoT button to your wifi network. 
<a href="https://aws.amazon.com/iotbutton/faq/"> <span style="background-color: black; color : white">https://aws.amazon.com/iotbutton/faq/</span> </a>

## Set up the rule that will fire when the IoT button is pressed

So far so good! Your button is configured and is able to send a message to AWS. Now, let’s configure a rule to fire on AWS when it receives a message from the button. We shall use AWS lambda (where our functions  run and may run in parallel). Create a function in AWS lambda and choose "Use a bluprint". Then search for "iot-button-email". Finally set up the necessary information and run the script. 

You should be able to see this in your email when you press your IoT button. Try it !

<img src="/assets/images/Iot-button/email.png" alt="email" style="width:1000px;height:500px;">

### Debugging
Go to `Monitoring` tab -> click `View logs in CloudWatch`

<img src="/assets/images/Iot-button/double_click.png" alt="double_click.png" style="width:1000px;height:500px;">

## Setting up IFTTT

1. Login to your IFTTT account and "connect" the Webhook Applet (<a href="https://ifttt.com/maker_webhooks"> <span style="background-color: black; color : white">https://ifttt.com/maker_webhooks</span> </a>) Once `connected`, take note of the key under which is the X's in: "URL: https://maker.ifttt.com/use/XXXXXXXXXXXXXXXX"
2. Create a recipe which can be triggered from the IoT button. The event name should be either `SINGLE`, `DOUBLE` or `LONG`. Finally, set any service for the `that` part of this IFTTT. 

Note: The recipes labeled as "SINGLE" "DOUBLE" "LONG" are triggered by single, double or long > 1.5s pressing the AWS IoT Button.

## Creating AWS Lambda for our IFTTT

1. Create a Lambda function using the same blueprint `iot-button-email`.
2. Copy the code provided below and paste into inline code editor.

**Remember to change `your private key here` to your own key.**

```javascript
var https = require('https');

exports.handler = function(event, context) {

    var body='';
    var jsonObject = JSON.stringify(event);

    // the post options
    var optionspost = {
        host: 'maker.ifttt.com',
        path: '/trigger/' + event.clickType + '/with/key/[your private key here]',
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        }
    };

    var reqPost = https.request(optionspost, function(res) {
        console.log("statusCode: ", res.statusCode);
        res.on('data', function (chunk) {
            body += chunk;
        });
        context.succeed('Thanks Marcus');
    });

    reqPost.write(jsonObject);
    reqPost.end();
};
```
I have configured the `that` portion of IFTTT to send me a message to my mobile device.
<img src="/assets/images/Iot-button/phone_message.jpg" alt="phone_message" style="width:450px;height:800px;">


That's it folks ! It is very easy to set up the AWS IoT button and IFTTT. You will be able to connect to any service within a click of a button. Enjoy creating interesting recipes for your button, the sky is your limit!

<!-- <a href="https://markdownmonster.west-wind.com/docs/_4xs10gaui.htm"> <span style="background-color: black; color : white">Marked text</span> </a> -->