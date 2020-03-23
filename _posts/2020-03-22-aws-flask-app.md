---

layout: post
title:  "Deploying a flask application to Elastic Beanstalk"
date:   2020-03-22 14:34:33 -0700
categories: update
---

### COVID 2019 News and Updates

Youtube links

Website explanation tour <https://youtu.be/vOiLUxaxc8k.html>

How to deploy the flask application  <https://youtu.be/67obIkAPmNs.html>

Ho to upload data from guthub <https://youtu.be/vTPN0FltBKA>

#### 1. Location of the URL for our site

```bash
http://flask-covid19-env.eba-wgnwr7pr.us-west-2.elasticbeanstalk.com/
```

![Screen Shot 2020-03-13 at 6.12.26 PM](/2020-03-22-aws-flask-app/site.png)

#### Login

![1](/2020-03-22-aws-flask-app/1.png)

#### Notify

![2](/2020-03-22-aws-flask-app/2.png)

#### Success

![3](/2020-03-22-aws-flask-app/3.png)

#### 2.Utilized Cloud Services

1. Website Hosting: AWS BeanStalk
2. AWS PostgreSQL
3. AWS DynamoDB
4. AWS SNS
5. Consumes Restfull API- NewsAPI
6. AWS S3

#### 3. Diagrams for Project

Flask Application

Flask application was deployed to an AWS Elastic Beanstalk environment. S3 bucket was used as cloud storage. Boto3  for Python was used to create, configure, and manage AWS services, such as EC2 and S3. All code was written on MacOS Linux machine with Atom text editor.

This apllication was developed by using the following tutorial on AWS site.

```html
https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create-deploy-python-flask.html
```

**Figure 1: Flow chart for the front page.**





![flowchart](/2020-03-22-aws-flask-app/flowchart.jpg)

**Figure 2: High Level Design for the website.**

![design2](/2020-03-22-aws-flask-app/design2.jpg)





As we can see from the figure above AWS Elastic BeanStalk offeres the loadbalancer. When trafiic grows to our website the systems automatically will be scaled to support the high demand.

Future work includes to automate the data load form Github to the PostgreSQL table. The idea is the data will be copied to S3 bucket and static url to S3 bucket will be used to upload data into PostgraSQL table. Currently all these done manually.

#### 4. Why I Chose AWS

I chose AWS because MackbookAir is my primary computer and I use Linux as an OS. I tried to develop a webpage using Visual Studio and Azure but Visual Studio for the Macbook Air didn't have most features that were described in the tutorials that I tried to follow. Also, when I tried to chose a language to code my first choice was JavaScript but pretty soon I didn't make any progress due to my javascript experience being limited. I finally decided to use Python as my coding language and AWS as a cloud service provider.

#### 5. How monitoring is done on the site

Elastick Beanstalk offers monitoring system to customers

<https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-health.html>

The document explains that developer can monitor environment health in the AWS management console.

AWS offers moitoring functionality.

![env0](/2020-03-22-aws-flask-app/env0.png)







![env](/2020-03-22-aws-flask-app/env.png)

Click on monitoring

Near the top we can see the stats for the environment. Also, overall environment health is shown in the graphs below.

![monitor](/2020-03-22-aws-flask-app/monitor.png)





#### 6. SLA Calculation

- Amazon DynamoDB Service Level Agreement: **99.99%**

* Amazon Messaging (SQS, SNS) Service Level Agreement: **99.9%**
*  NewsAPI: No uptime SLA as we are using free service. Assumed SLA **99.95%**

* Amazon PostgreSQL Service Level Agreement: **99.95%**

- AWS Elastic BeanStalk: **99.99% **

- Amazon S3 Service Level Agreement (“SLA”) : **99.99%**

**Case 1:**  Website

​	[Elastic BeanStalk] 99.99%  * [S3] 99.99% * [PostgreSQL]99.95% = **99.93%**

**Case 2:** Login and  Create Account Buttons

​	  [DynamoDB] 99.99% * [ Elastic BeanStalk]99.99%  = **99.98%**

**Case 3:** Notify

​	  [DynamoDB] 99.99% * [ Elastic BeanStalk]99.99% *  [ SNS]99.9%  = **99.88%**

#### 7. How the site will scale with load

According to the article in Medium, "Elastic Beanstalk Web servers are provisioned behind a load balancer and handle end-user requests, Elastic Beanstalk provides basic auto-scaling based on metrics collected from the underlying instances. We use Average CPU utilization as a metric for autoscaling."[1] As load grows AWS will start new instances for us and when the load goes down the service will turn off added instances and downsize according to the new load.



———————————— OTHER USEFUL INFO -------------------------------------------------------------------------------

#### Sending an SMS Message

AWS offers Simple Notification Services. On the website users will be able to sign up for text notifications

<https://docs.aws.amazon.com/sns/latest/dg/sms_publish-to-phone.html>

![sns](/2020-03-22-aws-flask-app/sns.png)

##### Dependencies

```bash
APScheduler==3.6.3
Flask-SQLAlchemy==2.4.1
SQLAlchemy==1.3.13
boto3==1.12.16
botocore==1.15.16
certifi==2019.11.28
chardet==3.0.4
Click==7.0
docutils==0.15.2
dominate==2.5.1
Flask==1.0.2
Flask-Bootstrap==3.3.7.1
Flask-Login==0.5.0
Flask-WTF==0.14.3
idna==2.9
itsdangerous==1.1.0
Jinja2==2.11.1
jmespath==0.9.5
MarkupSafe==1.1.1
newsapi-python==0.2.6
numpy==1.18.1
pandas==1.0.1
python-dateutil==2.8.1
pytz==2019.3
requests==2.23.0
s3transfer==0.3.3
six==1.14.0
tzlocal==2.0.0
urllib3==1.25.8
visitor==0.1.3
Werkzeug==1.0.0
WTForms==2.2.1
psycopg2-binary==2.8.4
```



Project folder info

![Screen Shot 2020-03-12 at 6.12.29 PM](/2020-03-22-aws-flask-app/folder.png)



Created flask-covid19-env

```bash
(virt) nps-MacBook-Air-2:eb-flask np$ eb create flask-covid19-env
Creating application version archive "app-200311_225521".
Uploading: [##################################################] 100% Done...
Environment details for: flask-covid19-env
  Application name: flask-covid19
  Region: us-west-2
  Deployed Version: app-200311_225521
  Environment ID: e-qqejrt2ykz
  Platform: arn:aws:elasticbeanstalk:us-west-2::platform/Python 3.6 running on 64bit Amazon Linux/2.9.6
  Tier: WebServer-Standard-1.0
  CNAME: UNKNOWN
  Updated: 2020-03-12 05:56:51.831000+00:00
Printing Status:
2020-03-12 05:56:50    INFO    createEnvironment is starting.
2020-03-12 05:56:52    INFO    Using elasticbeanstalk-us-west-2-051707507732 as Amazon S3 storage bucket for environment data.
2020-03-12 05:57:13    INFO    Created security group named: sg-0f76c5f0cf73c7d18
2020-03-12 05:57:15    INFO    Created load balancer named: awseb-e-q-AWSEBLoa-GTXGDRPFN6IN
2020-03-12 05:57:30    INFO    Created security group named: awseb-e-qqejrt2ykz-stack-AWSEBSecurityGroup-1GTN12UP63STW
2020-03-12 05:57:30    INFO    Created Auto Scaling launch configuration named: awseb-e-qqejrt2ykz-stack-AWSEBAutoScalingLaunchConfiguration-NNY2X3L9Y4WZ
2020-03-12 05:58:33    INFO    Created Auto Scaling group named: awseb-e-qqejrt2ykz-stack-AWSEBAutoScalingGroup-1GTYLLB04SGKE
2020-03-12 05:58:33    INFO    Waiting for EC2 instances to launch. This may take a few minutes.
2020-03-12 05:58:33    INFO    Created Auto Scaling group policy named: arn:aws:autoscaling:us-west-2:051707507732:scalingPolicy:73afe29f-58d2-4c1c-8e52-3254a47d93d1:autoScalingGroupName/awseb-e-qqejrt2ykz-stack-AWSEBAutoScalingGroup-1GTYLLB04SGKE:policyName/awseb-e-qqejrt2ykz-stack-AWSEBAutoScalingScaleUpPolicy-TB4DYH63FMW2
2020-03-12 05:58:33    INFO    Created Auto Scaling group policy named: arn:aws:autoscaling:us-west-2:051707507732:scalingPolicy:13a5cdf4-1cd1-4273-ac4a-f18fbc0232e1:autoScalingGroupName/awseb-e-qqejrt2ykz-stack-AWSEBAutoScalingGroup-1GTYLLB04SGKE:policyName/awseb-e-qqejrt2ykz-stack-AWSEBAutoScalingScaleDownPolicy-2Z20O5HD17XK
2020-03-12 05:58:33    INFO    Created CloudWatch alarm named: awseb-e-qqejrt2ykz-stack-AWSEBCloudwatchAlarmHigh-N3QVZXBFGZKS
2020-03-12 05:58:33    INFO    Created CloudWatch alarm named: awseb-e-qqejrt2ykz-stack-AWSEBCloudwatchAlarmLow-1UN0WN3JQR6WQ
2020-03-12 05:59:41    INFO    Application available at flask-covid19-env.eba-wgnwr7pr.us-west-2.elasticbeanstalk.com.
2020-03-12 05:59:41    INFO    Successfully launched environment: flask-covid19-env

```



#### Bibliography

1. Medium article about autoscaling <https://medium.com/tensult/elastic-beanstalk-with-autoscaling-81791a198095>
2. AWS CloudWatch <https://aws.amazon.com/cloudwatch/>
