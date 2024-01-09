This example assumes you have the AWS CLI configured with the necessary permissions and a basic understanding of AWS Lambda.

Create an IAM Role for Lambda:

i. Create an IAM role with the required permissions to describe and create EBS snapshots. Attach the following policy to your role:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:CreateSnapshot",
        "ec2:DescribeSnapshots",
        "ec2:DescribeVolumes"
      ],
      "Resource": "*"
    }
  ]
}
ii. Create a Lambda Function:

Create a Python script (e.g., backup_ebs.py) with the following code:

import boto3
import datetime

def lambda_handler(event, context):
    # Initialize the EC2 client
    ec2 = boto3.client('ec2')

    # Get a list of all EBS volumes in the region
    volumes = ec2.describe_volumes()['Volumes']

    # Create a snapshot for each volume
    for volume in volumes:
        volume_id = volume['VolumeId']
        snapshot_description = f"Backup_{volume_id}_{datetime.datetime.now().strftime('%Y%m%d%H%M%S')}"
        ec2.create_snapshot(VolumeId=volume_id, Description=snapshot_description)

    print("EBS backups completed successfully.")

Zip the script:
zip backup_ebs.zip backup_ebs.py

iii. Create a Lambda function using the AWS CLI:

aws lambda create-function \
    --function-name BackupEBS \
    --runtime python3.8 \
    --handler backup_ebs.lambda_handler \
    --role <your-iam-role-arn> \
    --zip-file fileb://backup_ebs.zip


iv. Set Up CloudWatch Events:
Create a CloudWatch Events rule to schedule the Lambda function:

aws events put-rule --name DailyBackupRule --schedule-expression "cron(0 0 * * ? *)"
Add permissions to allow CloudWatch Events to trigger the Lambda function:

aws lambda add-permission \
    --function-name BackupEBS \
    --statement-id DailyBackupRule \
    --action 'lambda:InvokeFunction' \
    --principal events.amazonaws.com \
    --source-arn arn:aws:events:<region>:<account-id>:rule/DailyBackupRule

v. Associate the rule with the Lambda function:
aws events put-targets --rule DailyBackupRule --targets "Id"="1","Arn"="arn:aws:lambda:<region>:<account-id>:function:BackupEBS"
Now, the Lambda function will be triggered daily to create snapshots for all EBS volumes in your AWS account. Adjust the script and configurations as needed based on your specific requirements.
