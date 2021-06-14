# Amazon MWAA at eprimo

This git repository contains all CloudFormation templates that were initially used to deploy Amazon MWAA at eprimo GmbH. Due to confidentiality reasons the default values in the parameter section were removed before publishing.

## Steps to create the intial Amazon MWAA setup
#### Step 1 - Create all necessary resources for Amazon MWAA
Firstly, we needed to create all necessary resources for the MWAA environment. At eprimo that included the following:
- An S3 Bucket that contains the Airflow DAGs, hooks, and sensors
- A CodePipeline that pulls files from a CodeCommit git repository and uploads them to S3
- A Lambda function that zips the custom hooks and sensors and uploads them to S3
- IAM roles for MWAA, CodePipeline and Lambda
- A security group for the MWAA resources

The corresponding (abstracted) CloudFormation template is named [_step_1_cfn_mwaa_base_resources.yaml_](https://github.com/tomasEprimo/mwaa_eprimo/blob/main/step_1_cfn_mwaa_base_resources.yaml).

#### Step 2 - Adjust the kms key policy
Disks of MWAA resources were supposed to be encrypted at-rest with KMS-CMK. This KMS key had to be identical to the key that is used to encrypt data stored in the S3 Bucket mentioned in step 1 to allow encrypted CloudWatch logs. To achieve this, the KMS key policy had to be changed accordingly

```
{
    "Sid": "Allow logs access",
    "Effect": "Allow",
    "Principal": {
        "Service": "logs.AWS-REGION.amazonaws.com"
    },
    "Action": [
        "kms:Encrypt*",
        "kms:Decrypt*",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:Describe*"
    ],
    "Resource": "*",
    "Condition": {
        "ArnLike": {
            "kms:EncryptionContext:aws:logs:arn": "arn:aws:logs:AWS-REGION:ACCOUNT-ID:*"
        }
    }
}
```
The dummy-values *AWS-REGION* and *ACCOUNT-ID* serve as placeholder for the actually to be inserted values.

#### Step 3 - Create the actual MWAA environment
Finally, we created the MWAA environment with the Apache Airflow version 1.16. The creation of the resources took about 20 minutes.
The resource section of the CloudFormation stack that creates the actual MWAA environment is shown in beginning of the following paragraph

The corresponding (abstracted) CloudFormation template is named [_step_3_cfn_mwaa_only.yaml_](https://github.com/tomasEprimo/mwaa_eprimo/blob/main/step_3_cfn_mwaa_environment.yaml).
