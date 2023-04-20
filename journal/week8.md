# Week 8 — Serverless Image Processing

## New Directory

Lets contain our cdk pipeline in a new top level directory called:

```sh
cd /workspace/aws-bootcamp-cruddur-2023
mkdir thumbing-serverless-cdk
```

## Install CDK globally

This is so we can use the AWS CDK CLI for anywhere.

```sh
npm install aws-cdk -g
```

We'll add the the install to our gitpod task file
```sh
  - name: cdk
    before: |
      npm install aws-cdk -g
```


## Initialize a new project

We'll initialize a new cdk project within the folder we created:

```sh
cdk init app --language typescript
```

## Add an S3 Bucket

Add the following code to your `thumbing-serverless-cdk-stack.ts`

```ts
import * as s3 from 'aws-cdk-lib/aws-s3';

const bucketName: string = process.env.THUMBING_BUCKET_NAME as string;

const bucket = new s3.Bucket(this, 'ThumbingBucket', {
  bucketName: bucketName,
  removalPolicy: cdk.RemovalPolicy.DESTROY,
});
```

```sh
export THUMBING_BUCKET_NAME="cruddur-thumbs"
gp env THUMBING_BUCKET_NAME="cruddur-thumbs"
```

- [Bucket Construct](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html)
- [Removal Policy](https://docs.aws.amazon.com/cdk/api/v1/docs/@aws-cdk_core.RemovalPolicy.html)

## Bootstrapping

> Deploying stacks with the AWS CDK requires dedicated Amazon S3 buckets and other containers to be available to AWS CloudFormation during deployment. 

```sh
cdk bootstrap "aws://$AWS_ACCOUNT_ID/$AWS_DEFAULT_REGION"
```

## Build

We can use build to catch errors prematurely.
This jsut builds tyescript

```sh
npm run build
```
install sharp and aws-sdk for 
```sh 
  cd aws/lambdas/process-images
  npm init -y
  npm i sharp
 npm i @aws-sdk/client-s3
```
## Synth

> the synth command is used to synthesize the AWS CloudFormation stack(s) that represent your infrastructure as code.

```sh
cdk synth
```


## Deploy

```sh
cdk deploy
```

## List Stacks

```sh
cdk ls
```
## Load Env Vars
  ```ts
const dotenv = require('dotenv');
dotenv.config();

const bucketName: string = process.env.THUMBING_BUCKET_NAME as string;
const folderInput: string = process.env.THUMBING_S3_FOLDER_INPUT as string;
const folderOutput: string = process.env.THUMBING_S3_FOLDER_OUTPUT as string;
const webhookUrl: string = process.env.THUMBING_WEBHOOK_URL as string;
const topicName: string = process.env.THUMBING_TOPIC_NAME as string;
const functionPath: string = process.env.THUMBING_FUNCTION_PATH as string;
console.log('bucketName',bucketName)
console.log('folderInput',folderInput)
console.log('folderOutput',folderOutput)
console.log('webhookUrl',webhookUrl)
console.log('topicName',topicName)
console.log('functionPath',functionPath)
```

## Create Bucket

```ts
import * as s3 from 'aws-cdk-lib/aws-s3';

const bucket = this.createBucket(bucketName)

createBucket(bucketName: string): s3.IBucket {
  const logicalName: string = 'ThumbingBucket';
  const bucket = new s3.Bucket(this, logicalName , {
    bucketName: bucketName,
    removalPolicy: cdk.RemovalPolicy.DESTROY,
  });
  return bucket;
}
```

## Create Lambda

```ts
import * as lambda from 'aws-cdk-lib/aws-lambda';

const lambda = this.createLambda(folderInput,folderOutput,functionPath,bucketName)

createLambda(folderIntput: string, folderOutput: string, functionPath: string, bucketName: string): lambda.IFunction {
  const logicalName = 'ThumbLambda';
  const code = lambda.Code.fromAsset(functionPath)
  const lambdaFunction = new lambda.Function(this, logicalName, {
    runtime: lambda.Runtime.NODEJS_18_X,
    handler: 'index.handler',
    code: code,
    environment: {
      DEST_BUCKET_NAME: bucketName,
      FOLDER_INPUT: folderIntput,
      FOLDER_OUTPUT: folderOutput,
      PROCESS_WIDTH: '512',
      PROCESS_HEIGHT: '512'
    }
  });
  return lambdaFunction;
}
```

## Create SNS Topic

```ts
import * as sns from 'aws-cdk-lib/aws-sns';

const snsTopic = this.createSnsTopic(topicName)

createSnsTopic(topicName: string): sns.ITopic{
  const logicalName = "Topic";
  const snsTopic = new sns.Topic(this, logicalName, {
    topicName: topicName
  });
  return snsTopic;
}
```

## Create an SNS Subscription

```ts
import * as s3n from 'aws-cdk-lib/aws-s3-notifications';

this.createSnsSubscription(snsTopic,webhookUrl)

createSnsSubscription(snsTopic: sns.ITopic, webhookUrl: string): sns.Subscription {
  const snsSubscription = snsTopic.addSubscription(
    new subscriptions.UrlSubscription(webhookUrl)
  )
  return snsSubscription;
}
```

## Create S3 Event Notification to SNS

```ts
this.createS3NotifyToSns(folderOutput,snsTopic,bucket)

createS3NotifyToSns(prefix: string, snsTopic: sns.ITopic, bucket: s3.IBucket): void {
  const destination = new s3n.SnsDestination(snsTopic)
  bucket.addEventNotification(
    s3.EventType.OBJECT_CREATED_PUT, 
    destination,
    {prefix: prefix}
  );
}
```

## Create S3 Event Notification to Lambda

```ts
this.createS3NotifyToLambda(folderInput,laombda,bucket)

createS3NotifyToLambda(prefix: string, lambda: lambda.IFunction, bucket: s3.IBucket): void {
  const destination = new s3n.LambdaDestination(lambda);
    bucket.addEventNotification(s3.EventType.OBJECT_CREATED_PUT,
    destination,
    {prefix: prefix}
  )
}
```

## Create Policy for Bucket Access

```ts
const s3ReadWritePolicy = this.createPolicyBucketAccess(bucket.bucketArn)
```

## Create Policy for SNS Publishing

```ts
const snsPublishPolicy = this.createPolicySnSPublish(snsTopic.topicArn)
```

## Attach the Policies to the Lambda Role

```ts
lambda.addToRolePolicy(s3ReadWritePolicy);
lambda.addToRolePolicy(snsPublishPolicy);
```
## Serving Avatars via CloudFront

Amazon CloudFront is designed to work seamlessly with S3 to serve your S3 content in a faster way. Also, using CloudFront to serve s3 content gives you a lot more flexibility and control. To create a CloudFront distribution, a certificate in the `us-east-1` zone for `*.<your_domain_name>` is required. If you don't have one yet, create one via AWS Certificate Manager, and click "Create records in Route 53" after the certificate is issued.

Create a distribution by:

- set the Origin domain to point to `assets.<your_domain_name>`
- choose Origin access control settings (recommended) and create a control setting
- select Redirect HTTP to HTTPS for the viewer protocol policy
- choose CachingOptimized, CORS-CustomOrigin as the optional Origin request policy, and SimpleCORS as the response headers policy
- set Alternate domain name (CNAME) as `assets.<your_domain_name>`
- choose the previously created ACM for the Custom SSL certificate.

Remember to copy the created policy to the `assets.<your_domain_name>` bucket by editing its bucket policy.

In order to visit `https://assets.<your_domain_name>/avatars/data.jpg` to see the processed image, we need to create a record via Route 53:

- set record name as `assets.<your_domain_name>`
- turn on alias, route traffic to alias to CloudFront distribution
- - in my case, you can see my profile at https://assets.fathytekitek.fr/avatars/data.jpg
- Since we don't use versioned file names for a user's avatar, CloudFront edge caches old avatar. Until the old one expires, you will not immediately see the new avatar after updating the profile. Therefore, we need to [invalidate files](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Invalidation.html) by creating an invalidation:

- go to the distribution we created
- under the Invalidations tab, click create
- add object path `/avatars/*`

This ensures that CloudFront will always serve the latest avatar uploaded by the user.

## DB Migration

Since our previous postgres database didn't have the column for saving bio, migration is required. We also need to update some backend scripts in order to let users edit bio and save the updated bio in the database.

Create an empty `backend-flask/db/migrations/.keep`, and an executable script `bin/generate/migration` as seen in [code]## DB Migration

Since our previous postgres database didn't have the column for saving bio, migration is required. We also need to update some backend scripts in order to let users edit bio and save the updated bio in the database.

Create an empty `backend-flask/db/migrations/.keep`, and an executable script `bin/generate/migration` as seen in [code] . Run `./bin/generate/migration add_bio_column`, a python script such as `backend-flask/db/migrations/1681742424_add_bio_column.py` will be generated. Edit the generated python script with SQL commands as seen in [code].

Update `backend-flask/db/schema.sql`, and update `backend-flask/lib/db.py` with verbose option.

Create executable scripts `bin/db/migrate` and `bin/db/rollback` as seen in [repo] . If we run `./bin/db/migrate`, a new column called bio will be created in the db table of `users`.

## Implement Avatar Uploading

Firstly we need to create an API endpoint, which invoke a presigned URL like `https://<API_ID>.execute-api.<AWS_REGION>.amazonaws.com`. This presigned URL can give access to the S3 bucket (`beici-cruddur-uploaded-avatars` in my case), and can deliver the uploaded image to the bucket.

We will call `https://<API_ID>.execute-api.<AWS_REGION>.amazonaws.com/avatars/key_upload` to do the upload, where the `/avatars/key_upload` resource is manipulated by the `POST` method. We will also create a Lambda function named `CruddurAvatarUpload` to decode the URL and the request. In addition, we need to implement authorization with another Lambda function named `CruddurApiGatewayLambdaAuthorizer`, which is important to control the data that is allowed to be transmitted from our gitpod workspace using the APIs.

To successfully implement above setups:

- in `aws/lambdas/cruddur-upload-avatar/`, create a basic `function.rb` and run `bundle init`; edit the generated `Gemfile`, then run `bundle install` and `bundle exec ruby function.rb`; a presigned url can be generated for local testing. The actual `function.rb` used in `CruddurAvatarUpload` is shown as in [this code].
- in `aws/lambdas/lambda-authorizer/`, create `index.js`, run `npm install aws-jwt-verify --save`, and download everything in this folder into a zip file (you can zip by command `zip -r lambda_authorizer.zip .`), which will be uploaded into `CruddurApiGatewayLambdaAuthorizer`.

At AWS Lambda, create the corresponding two functions:

1. `CruddurAvatarUpload`

   - code source as seen in `aws/lambdas/cruddur-upload-avatar/function.rb` with your own gitpod frontend URL as `Access-Control-Allow-Origin`
   - rename Handler as function.handler
   - add environment variable `UPLOADS_BUCKET_NAME`
   - create a new policy `PresignedUrlAvatarPolicy` as seen in `aws/policies/s3-upload-avatar-presigned-url-policy.json` ([code]), and then attach this policy to the role of this Lambda

2. `CruddurApiGatewayLambdaAuthorizer`

   - upload `lambda_authorizer.zip` into the code source
   - add environment variables `USER_POOL_ID` and `CLIENT_ID`

Different from Andrew's video and his codes, I don't have a layer of JWT cause I passed the JWT sub from `CruddurApiGatewayLambdaAuthorizer` to `CruddurAvatarUpload` ([reference](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-lambda-authorizer.html)).

At AWS S3, update the permissions of `beici-cruddur-uploaded-avatars` by editing the CORS configuration as seen in `aws/s3/cors.json` ([code]).

At AWS API Gateway, create `api.<domain_name>` (in my case `api.beici-demo.xyz`), create two routes:

- `POST /avatars/key_upload` with authorizer `CruddurJWTAuthorizer` which invoke Lambda `CruddurApiGatewayLambdaAuthorizer`, and with integration `CruddurAvatarUpload`
- `OPTIONS /{proxy+}` without authorizer, but with integration `CruddurAvatarUpload`

Noted that we don't need to configure CORS at API Gateway. If you did before, click "Clear" to avoid potential CORS issues.

## Double Check Environment Variables

There are some environment variables and setups worth double checking:

- `function.rb` in `CruddurAvatarUpload`: set `Access-Control-Allow-Origin` as your own frontend URL.
- `index.js` in `CruddurApiGatewayLambdaAuthorizer`: make sure that token can be correctly extracted from the authorization header.
- Environment variables in the above two Lambdas were added.
- `erb/frontend-react-js.env.erb`: `REACT_APP_API_GATEWAY_ENDPOINT_URL` equals to the Invoke URL shown in the API Gateway.
- `frontend-react-js/src/components/ProfileForm.js`: `gateway_url` and `backend_url` are correctly set.
- Pay attention to variable name inconsistency in some scripts, e.g., `cognito_user_uuid` vs. `cognito_user_id`.). Run `./bin/generate/migration add_bio_column`, a python script such as `backend-flask/db/migrations/1681742424_add_bio_column.py` will be generated. Edit the generated python script with SQL commands as seen in [code].

Update `backend-flask/db/schema.sql`, and update `backend-flask/lib/db.py` with verbose option.

Create executable scripts `bin/db/migrate` and `bin/db/rollback` as seen in [repo] . If we run `./bin/db/migrate`, a new column called bio will be created in the db table of `users`.

## Implement Avatar Uploading

Firstly we need to create an API endpoint, which invoke a presigned URL like `https://<API_ID>.execute-api.<AWS_REGION>.amazonaws.com`. This presigned URL can give access to the S3 bucket (`beici-cruddur-uploaded-avatars` in my case), and can deliver the uploaded image to the bucket.

We will call `https://<API_ID>.execute-api.<AWS_REGION>.amazonaws.com/avatars/key_upload` to do the upload, where the `/avatars/key_upload` resource is manipulated by the `POST` method. We will also create a Lambda function named `CruddurAvatarUpload` to decode the URL and the request. In addition, we need to implement authorization with another Lambda function named `CruddurApiGatewayLambdaAuthorizer`, which is important to control the data that is allowed to be transmitted from our gitpod workspace using the APIs.

To successfully implement above setups:

- in `aws/lambdas/cruddur-upload-avatar/`, create a basic `function.rb` and run `bundle init`; edit the generated `Gemfile`, then run `bundle install` and `bundle exec ruby function.rb`; a presigned url can be generated for local testing. The actual `function.rb` used in `CruddurAvatarUpload` is shown as in [this code].
- in `aws/lambdas/lambda-authorizer/`, create `index.js`, run `npm install aws-jwt-verify --save`, and download everything in this folder into a zip file (you can zip by command `zip -r lambda_authorizer.zip .`), which will be uploaded into `CruddurApiGatewayLambdaAuthorizer`.

At AWS Lambda, create the corresponding two functions:

1. `CruddurAvatarUpload`

   - code source as seen in `aws/lambdas/cruddur-upload-avatar/function.rb` with your own gitpod frontend URL as `Access-Control-Allow-Origin`
   - rename Handler as function.handler
   - add environment variable `UPLOADS_BUCKET_NAME`
   - create a new policy `PresignedUrlAvatarPolicy` as seen in `aws/policies/s3-upload-avatar-presigned-url-policy.json` ([code]), and then attach this policy to the role of this Lambda

2. `CruddurApiGatewayLambdaAuthorizer`

   - upload `lambda_authorizer.zip` into the code source
   - add environment variables `USER_POOL_ID` and `CLIENT_ID`

Different from Andrew's video and his codes, I don't have a layer of JWT cause I passed the JWT sub from `CruddurApiGatewayLambdaAuthorizer` to `CruddurAvatarUpload` ([reference](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-lambda-authorizer.html)).

At AWS S3, update the permissions of `beici-cruddur-uploaded-avatars` by editing the CORS configuration as seen in `aws/s3/cors.json` ([code]).

At AWS API Gateway, create `api.<domain_name>` (in my case `api.beici-demo.xyz`), create two routes:

- `POST /avatars/key_upload` with authorizer `CruddurJWTAuthorizer` which invoke Lambda `CruddurApiGatewayLambdaAuthorizer`, and with integration `CruddurAvatarUpload`
- `OPTIONS /{proxy+}` without authorizer, but with integration `CruddurAvatarUpload`

Noted that we don't need to configure CORS at API Gateway. If you did before, click "Clear" to avoid potential CORS issues.

## Double Check Environment Variables

There are some environment variables and setups worth double checking:

- `function.rb` in `CruddurAvatarUpload`: set `Access-Control-Allow-Origin` as your own frontend URL.
- `index.js` in `CruddurApiGatewayLambdaAuthorizer`: make sure that token can be correctly extracted from the authorization header.
- Environment variables in the above two Lambdas were added.
- `erb/frontend-react-js.env.erb`: `REACT_APP_API_GATEWAY_ENDPOINT_URL` equals to the Invoke URL shown in the API Gateway.
- `frontend-react-js/src/components/ProfileForm.js`: `gateway_url` and `backend_url` are correctly set.
- Pay attention to variable name inconsistency in some scripts, e.g., `cognito_user_uuid` vs. `cognito_user_id`.