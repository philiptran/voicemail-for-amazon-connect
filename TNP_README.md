# Rebuild the project and update to existing Lambda functions

Note: this is just a quick hack to re-build the code and update the 2 key Lambda functions: `XX-ContactVoicemailStreamXXX` and `XX-KvsProcessRecordingXX` that are already deployed in your AWS account using the CloudFormation stack.

## Prerequisites

Clone the GitHub repo

```
git clone https://github.com/philiptran/voicemail-for-amazon-connect.git
```

Install NodeJS libs and tools

```
cd voicemail-for-amazon-connect
cd source/aws-connect-vm-serverless/
npm install
npm install --only=dev
serverless -v
```

Install Java and Maven for your IDE

1. For Cloud9 IDE, refer to the instructions here: https://docs.aws.amazon.com/cloud9/latest/user-guide/sample-java.html
2. Verify

```
java --version
mvn --version
```

## Build the project

Check that you are still in the `voicemail-for-amazon-connect/source/aws-connect-vm-serverless/` folder. Use `cd` command to go to the `aws-connect-vm-serverless` folder if required.

Run `npm run build` to build the Java code and generate the bundle `aws-connect-vm-java.jar` file in the `target` folder. This command also generates the minimised packages for NodeJS codes in the `handler` folder.

```
npm run build
```

Run `serverless package` to package the NodeJS code and generate the `aws-connect-vm.zip` file in a hidden folder `.serverless`.

```
serverless package
```

## Deploy the changes to your existing Lambda functions

**Important**: Make sure to test the changes in your UAT environment before promoting the changes to Production!

Update the first Lambda function `KvsProcessRecording`:

1. Open the Lambda console and find the Lambda function containing this string `KvsProcessRecording`.
2. Click the Lambda function to open it.
3. Under the Code tab, click `Upload from` and choose `.zip or .jar file`.
4. In the pop-up window, click `Upload` and choose the `aws-connect-vm-java.jar` from the `target` folder created by your build above.
5. Click `Save` to save the changes.

Update the second Lambda function `ContactVoicemailStream`:

1. Open the Lambda console and find the Lambda function containing this string `ContactVoicemailStream`.
2. Click the Lambda function to open it.
3. Under the Code tab, click `Upload from` and choose `.zip file`.
4. In the pop-up window, click `Upload` and choose the `aws-connect-vm.zip` from the `.serverless` folder created by your build above. If you cannot see the hidden `.serverless` folder, use the File Explorer to copy the `aws-connect-vm.zip` to your Downloads folder. Repeat step 3 to upload the zip file from the `Downloads` folder instead.
5. Click `Save` to save the changes.


## Add the `contactFlowName` contact attribute to your contact flow that uses the Voicemail feature

1. Open the contact flow that uses the customer queue for Voicemail
2. Use `Set contact attributes` block to add a new User defined attribute as follow.

| Key             | Value            |
| --------------- | ---------------- |
| contactFlowName | [your own value] |

3. Click `Save` to save the changes.
4. Click `Save` and then click `Publish` to publish the changes to the contact flow.

## Verification and testing

1. Make a test call and leave a voicemail message.
2. Check the CloudWatch logs for the `ContactVoicemailStream` function.
3. Find the `mailOptions` log entry to see the details of the email notification that is sent to the manager's email address.
4. Manager should have also received an email notification if the email address has been verified and confirmed in Amazon SES.




