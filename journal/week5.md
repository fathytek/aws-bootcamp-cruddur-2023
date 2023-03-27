# Week 5 — DynamoDB and Serverless Caching

# Security in DynamoDB

DynamoDB is a None Relational Database and is used for high performace applications at any scale.

Below some use cases of DynamoDB:

![DynamoDB use cases](
https://cdn.sanity.io/images/hgftikht/production/f9381ef455f0c2a07601a6b55113c44e1acae538-2060x1150.png?w=1920&h=1072&fit=crop&fm=webp)


How to access DynamoDB

![DynamoDB via Internet Gateway](https://docs.aws.amazon.com/images/amazondynamodb/latest/developerguide/images/ddb-no-vpc-endpoint.png)


![DynamoDB via Internet Gateway](https://docs.aws.amazon.com/images/vpc/latest/privatelink/images/without-gateway-endpoints.png)

In these 2 diagrams, the communication with dynamodb goes outside the aws through the internet and reaches dynamodb endpoint. This is not a good practice first from a security and costs perspective.

![DynamoDB via VPC ENDPOINT](https://docs.aws.amazon.com/images/amazondynamodb/latest/developerguide/images/ddb-yes-vpc-endpoint.png)

![DynamoDB vi GATEWAY ENDPOINT](https://docs.aws.amazon.com/images/vpc/latest/privatelink/images/gateway-endpoints.png)

In these 2 diagrams, the communication with dyanomodb is within the aws network and not through the public internet.

## Best Practice

### Aws Prospective
- Use VPC Endpoint to create a private connection from your application to the dynamodb. This helps prevent unauthorised access  to your instance from public internet
- Compliance Standard
- Dynamodb should only be in the AWS region that you are legally allowed to be holding user data in.
- Amazon Organization SCP to manage permission to do the operation
- AWS Cloudtrail helps to monitor and trigger alerts on malicious DyanmoDB behaviour.
- AWS Config Rule (Regional Service)

### Application Prospective
- Dynamo DB to use appropriate Authentication such as IAM Roles or AWS Cognito Identity Pool (Temporary credentials) rather than permanent credential such as IAM users or group
- Dyanomodb User Lifecycle Management
- AWS IAM role instead of individual users to access and manage dynamodb
 DAX Service (IAM) role to have read-only access to dynamodb
 - Do not access dynamodb through the internet, use site-to-site or Direct Connect to access dynamodb from the on-premise.
 - Client Side encryption for sensitive information on dynamodb (recommended by amazon)






Reference
- Contino.Io
- AWS Documentation

## DynamoDB Bash Scripts

```sh
./bin/ddb/schem-load
```


## The Boundaries of DynamoDB

- When you write a query you have provide a Primary Key (equality) eg. pk = 'andrew'
- Are you allowed to "update" the Hash and Range?
  - No, whenever you change a key (simple or composite) eg. pk or sk you have to create a new item.
    - you have to delete the old one
- Key condition expressions for query only for RANGE, HASH is only equality 
- Don't create UUID for entity if you don't have an access pattern for it


3 Access Patterns

## Pattern A  (showing a single conversation)

A user wants to see a list of messages that belong to a message group
The messages must be ordered by the created_at timestamp from newest to oldest (DESC)

```sql
SELECT
  messages.uuid,
  messages.display_name,
  messages.message,
  messages.handle,
  messages.created_at -- sk
FROM messages
WHERE
  messages.message_group_uuid = {{message_group_uuid}} -- pk
ORDER BY messages.created_at DESC
```

> message_group_uuid comes from Pattern B

## Pattern B (list of conversation)

A user wants to see a list of previous conversations.
These conversations are listed from newest to oldest (DESC)
We want to see the other person we are talking to.
We want to see the last message (from whomever) in summary.

```sql
SELECT
  message_groups.uuid,
  message_groups.other_user_uuid,
  message_groups.other_user_display_name,
  message_groups.other_user_handle,
  message_groups.last_message,
  message_groups.last_message_at
FROM message_groups
WHERE
  message_groups.user_uuid = {{user_uuid}} --pk
ORDER BY message_groups.last_message_at DESC
```

> We need a Global Secondary Index (GSI)

## Pattern C (create a message)

```sql
INSERT INTO messages (
  user_uuid,
  display_name,
  handle,
  creaed_at
)
VALUES (
  {{user_uuid}},
  {{display_name}},
  {{handle}},
  {{created_at}}
);
```

## Pattern D (update a message_group for the last message)

When a user creates a message we need to update the conversation
to display the last message information for the conversation

```sql
UPDATE message_groups
SET 
  other_user_uuid = {{other_user_uuid}}
  other_user_display_name = {{other_user_display_name}}
  other_user_handle = {{other_user_handle}}
  last_message = {{last_message}}
  last_message_at = {{last_message_at}}
WHERE 
  message_groups.uuid = {{message_group_uuid}}
  AND message_groups.user_uuid = {{user_uuid}}
```
