# Week 9 — CI/CD with CodePipeline, CodeBuild and CodeDeploy

### CI CD Piple Security Explained for AWS

**What is a CI/CD**
- CI : Continuous Integration
- CD : Continuous Deployment
- Deploy code into production many time without manually building and testing

What AWS services can help to achieve CI/CD
- CodeCommit
- CodeBuild
- CodeDeploy
- CodePipeline - CI/CD flow Orchestrator

![image](https://user-images.githubusercontent.com/32872009/235112693-db0a8a76-2312-4791-92fc-46e37a569d0e.png)

you can use others Code Repo like GITLab, GITHub, BitBucket

**Simple CI/CD Pipeline Architecture**

![image](https://user-images.githubusercontent.com/32872009/235112557-973a8e59-6078-41f8-96f2-f8ff7f1c41d3.png)


**OWASP Top 10 CI/CD**
here the top ten CI/CD Security Risks
- CICD-SEC1 : Insufficient Flow Control Mechanims
- CICD-SEC2 : Inadequate Identity and Access Management
- CICD-SEC3 : Dependency Chanin Abuse
- CICD-SEC4 : Poisoned Pipeline Execution (PPE)
- CICD-SEC5 : Insufficient PBAC (Pipeline-Based Access Controls)
- CICD-SEC6 : Insufficient Credential Hygiene
- CICD-SEC7 : Insecure System Configuration
- CICD-SEC8 : Ungoverned Usage of 3rd Party Services
- CICD-SEC9 : Improper Artifact Integrity Validation
- CICD-SEC10: Insufficient Logging and Visibility

**Amazon Secrutiy Best Practices**

*AWS - Best Practices*
- Compliance standard is what your business requires from a CI/CD service and is available in the region you need to operate in
- Amazon Organizations SCP - to restrict actions like creation, deletion, modification of production CI/CD pipeline services etc
- AWS CloudTrail is enabled & monitored to trigger alerts for malicious activities e.g changes to Production CodePipeline, etc.
- GuardDuty is enabled for monitoring suspicious DNS comms(e.g Crypto-mining etc) and automated for auto-remediation
- AWS Config Rules is enabled in the account and region of CodeBuild  Performance pack for any other CI/CD service

*Application - Best Practices*
- Access control - Roles or IAM Users for making changes in Amazon CICD services especially production related repositories, pipeline, build services).
- Security of the CI/CD - source control, secret management, container registry, CI/CD service (if not the AWS Service), IAM etc
- Security in the CI/CD Pipeline - Code Security Best Practices - SCa, SAST, Secret Scanner, DAST implemented  in the CI/CD pipeline
- Security of the CI/CD Pipeline entry point e.g - no bypass of CI/CD to make prodcution changes
- Enable Encryption in Transit using TLS/SSL certification eg HTTPS
- Only use Trusted Source Control for sending changes to CI/CD pipeline
- Develop Process for continuously verifiying if there is a change that may compromise the know state of a CI/CD pipeline

## AWS CodeBuild

Create a build project:

- name as `cruddur-backend-flask-bake-image`, enable build badge
- source:
  - choose source provider as GitHub, repository in my GitHub account, select the `cruddur` repo, set source version to `prod`
  - select rebuild every time a code change is pushed to this repository, select single build, select event type as `PULL_REQUEST_MERGED`
- environment:
  - select managed image, select operating system as Amazon Linux 2, select standard runtime, select the latest image (4.0), select environment type as Linux, tick privileged
  - create a new service role automatically named as `codebuild-cruddur-backend-flask-bake-image-service-role`
  - decrease timeout to 20 min, don't select any certificate nor VPC, select compute as 3 GB memory and 2 vCPUs
- use a buildspec file `backend-flask/buildspec.yml`
- no artifects
- select cloudwatch logs, set group name as `/cruddur/build/backend-flask`, stream name as `backend-flask`

Then click "Start build" (or triggered by a merge to the `prod` branch). If succeeded, you can check the build history for details.
![image](https://user-images.githubusercontent.com/32872009/235320396-0b3b17a4-b265-4684-80b3-56437a0ea5a0.png)

![image](https://user-images.githubusercontent.com/32872009/235320359-aefb16ba-9c9d-4bbb-b0de-f321fe26f68f.png)

## AWS CodePipeline

Create a pipeline:

- name as `cruddur-backend-fargate`, allow to create a new service role automatically named as `AWSCodePipelineServiceRole-us-east-1-cruddur-backend-fargate`, select default location and default managed key in advanced settings
- source stage from GitHub (Version 2), click "Connect to GitHub", set connection name as `cruddur`, install a new app, select the cruddur repo, in the end finish "Connect to GitHub" and back to the pipeline page
- select the cruddur repo and select branch `prod`, select "start the pipeline on source code change" and default output artifact format
- for build stage, select AWS CodeBuild as build provider, select your region, select the newly created project `cruddur-backend-flask-bake-image`
- for deploy stage, select ECS as deploy provide, choose `cruddur` cluster, `backend-flask` service
  