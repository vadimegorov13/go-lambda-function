# Simple Go lambda function

## Deploying

1. Install the dependencies

```bash
go mod tidy
```
This will install github.com/aws/aws-lambda-go

2. Connect AWS account through the CLI

```bash
export AWS_ACCESS_KEY_ID=<YOUR_ACCESS_KEY_ID>
export AWS_SECRET_ACCESS_KEY=<YOUR_SECRET_ACCESS_KEY>
export AWS_SESSION_TOKEN=<YOUR_SESSION_TOKEN>
```
Connects your CLI to your AWS account using your access key, secret key, and session token.

3. Create new role for lambda

```bash
export ROLE_NAME="lambda-ex"

aws iam create-role --role-name $ROLE_NAME \
--assume-role-policy-document file://trust-policy.json
```

4. Assign policy arn for the role

```bash
aws iam attach-role-policy --role-name $ROLE_NAME \
--policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```
Assigns a policy to the role that grants the minimum permissions required to execute the lambda function.

5. Build function

```bash
GOOS=linux GOARCH=amd64 go build -o main
```
This will produce an executable file named main that is compatible with Linux x86_64 architecture, which you can then deploy to your AWS Lambda function.

6. Zip main

```bash
zip function.zip main
```
Creates a compressed zip file that includes the main executable.

7. Create lambda function

```bash
export FUNCTION_NAME="go-lambda-function"
export IAM_ID=<YOUR_IAM_ID>

aws lambda create-function --function-name $FUNCTION_NAME \
--zip-file fileb://function.zip --handler main --runtime go1.x \
--role arn:aws:iam::$IAM_ID:role/$ROLE_NAME
```
Creates a new lambda function and uploads the zip file containing the main executable.

## Test

Test the lambda function by invoking it with a sample payload and writing the output to a file.

```bash
# Functions list
aws lambda list-functions

# Invoke function
aws lambda invoke --function-name $FUNCTION_NAME \
--cli-binary-format raw-in-base64-out \
--payload '{"name": "Vadim", "age": 22}' output.txt
```

## Clean up

1. Delete the lambda function.
```bash
aws lambda delete-function --function-name $FUNCTION_NAME
```

2. To delete the role, first you need to detach all policies
```bash
aws iam detach-role-policy --role-name $ROLE_NAME \
--policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

3. Delete role created earlier
```bash
aws iam delete-role --role-name $ROLE_NAME
```

