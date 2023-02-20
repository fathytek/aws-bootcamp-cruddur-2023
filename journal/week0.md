# Prerequisite
Before starting this bootcamp, please follow the palylisting below :
https://www.youtube.com/playlist?list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv
You need :
  - Create your github account and use the template of Andrew's repository
  - Create your gitpod account and install the extension in your browser. a gitpot button will appear in you github repo
  - Create your Github Codespace
  - Create your AWS account ( credit card is mandatory) and select the free tier. you must activate your MFA and destroy all Access Key and please avoid to publish them in Github
  - Create you Lucichart account. you will be able to create your digram.
  - Create your Honeycomb.io account
  - Create your RollBar account
  -![image](https://user-images.githubusercontent.com/32872009/219142347-fbe80a9e-2110-4fb3-99f6-b198b382b601.png)

 Big thanks for Shala and her smile for all the youtube video


# Week 0 — Billing and Architecture

1 -Installing AWS CLI

First I installed AWS CLI on my laptop ( Windows OS based) by following the guid in AWS documentation.


2-  Create Admin User


 in this section, I create a group with IAM with admin right. I rename this group as admin
 
 ![image](https://user-images.githubusercontent.com/32872009/219869390-5d326273-63c9-4eae-a976-0d07d32d9e3c.png)
 
 I give the permission : AdministratorAcces
 
 Once done, In the Users panel, I create the user AWS-BOOT-CAMP user and 
 
 ![image](https://user-images.githubusercontent.com/32872009/219869535-3e24d937-78c4-48ed-acf1-8caa01bab607.png)

I assigned to the group admin 

![image](https://user-images.githubusercontent.com/32872009/219869547-078a0eb8-aa24-4569-a1b5-41e4ebcaa0ab.png)

I tagged my resource

![image](https://user-images.githubusercontent.com/32872009/219870525-8e7f179d-bff1-4f3b-a151-f52bfa708798.png)

and the result is my AWS-BOOT-CAMP was created as follow

![image](https://user-images.githubusercontent.com/32872009/219870541-fc371639-4fe6-4272-acfe-fd28ff859780.png)


Once the user was created, I enabled the access to the console and I activate the MFA feature.


3 - Generate AWS Credentials

With my user freshly created I connected to the console. I select the IAM service and I choose AWS-BOOT-CAMP. 

![image](https://user-images.githubusercontent.com/32872009/219870773-c11b80bc-5f38-408b-a519-6fc3c9bb7604.png)


I the tab, I select Security credentials I click on the button Create access key in the Access Key section

![image](https://user-images.githubusercontent.com/32872009/219870851-be7a2265-13a5-4d6e-94b1-f996d8a9a03b.png)


once create we need to set the differents variable for the dedicated environments ( Current Terminal & Gitpod)
```
export AWS_ACCESS_KEY_ID=""
export AWS_SECRET_ACCESS_KEY=""
export AWS_DEFAULT_REGION=us-east-1
```
For Gitpod enn you need to launch these commands
```
gp env AWS_ACCESS_KEY_ID=""
gp env AWS_SECRET_ACCESS_KEY=""
gp env AWS_DEFAULT_REGION=us-east-1
```


4 - CloudShell

I connected to the AWS console with the AWS-BOOT-CAMP user and I activate my CloudShell

![image](https://user-images.githubusercontent.com/32872009/219871800-1b9dbe4e-27ea-49e6-8032-2e4b569873ef.png)


very easy part.

5 - Billing Alarm

for this section, we will use the AWS Cli to create the alarm as shown in Andrew Video. as prerequisite for the alarm, we need a sns topic first and after we can build the alarm.

  Create SNS Topic
  
  I will create the SNS topic using the following command
```
aws sns create-topic --name my-first-alarm
```
once the TopicArn is returned, you can create a subscription using it with your email
```
aws sns subscribe \
    --topic-arn TopicARN \
    --protocol email \
    --notification-endpoint your@email.com
```
  chekc you email box for the confirmation
  
  Create Alarm
  
 you can use the create alarm via AWS CLI
 ```
 aws cloudwatch put-metric-alarm --cli-input-json file://aws/json/alarm_config.json
```
6 - Budgets

I logged as root in my account and I create as suggested by Andrew 2 alarms. the first one with zero spend budget and the second for 10 dollars monthly cost budget.
Here below the steps I follow to create the first one. I used AWS console to create this budget. I choose Use a template (simplified) and I select from Templates - new Zero spend budget

![image](https://user-images.githubusercontent.com/32872009/219864458-bf32cd74-deb3-4e95-bf81-146e10f8f936.png)

here the result of my creation 

![image](https://user-images.githubusercontent.com/32872009/219864534-497ae179-04c1-4dbd-80f3-85415633ad53.png)

for the second budget here some screenshoot which describe my progression

![image](https://user-images.githubusercontent.com/32872009/219864609-e6f382f9-304e-4956-b2ce-de6d7bcf575a.png)

i rename this budget as My Monthly Cost Budget 20 Dollars

![image](https://user-images.githubusercontent.com/32872009/219864623-ad23d2db-cdea-4eb9-9bed-d395ef1ae66e.png)


7- Lucid Diagram

I tried to follow the video and I reproduced the cruddur conceptual Diagram
here the link : 
https://lucid.app/lucidchart/cd3eb922-36c2-400c-a81f-0dee949c466d/edit?beaconFlowId=11055289816A723C&invitationId=inv_a1171102-8146-486e-afe4-8db78eff116b&page=0_0#


![image](https://user-images.githubusercontent.com/32872009/219875589-d49f25ba-ef2a-45d5-98e9-39f864632d92.png)


8 - Lucid Logical Diagram

Here the Logical Diagram of Cruddur which represents :
. Application using micro services
. The frontend is in JS and the backend is in Python
. Using api to communicate
. Authentication using Cognito
. Momento as a third party caching system

![image](https://user-images.githubusercontent.com/32872009/220203387-e37bdb70-02ab-41ce-bcf9-dff8531a3229.png)


https://lucid.app/lucidchart/invitations/accept/inv_c760eee5-7bf3-4cda-94ed-2640b3302434




