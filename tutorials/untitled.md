---
description: Setting up Cluster backup using Velero on AWS
---

# Velero

### **Create S3 bucket**

Velero requires an object storage bucket to store backups in, preferrably unique to a single Kubernetes cluster \(see the [FAQ](https://velero.io/docs/v1.1.0/faq/) for more details\). Create an S3 bucket, replacing placeholders appropriately:

```text
BUCKET=<YOUR_BUCKET>
REGION=<YOUR_REGION>
aws s3api create-bucket \
    --bucket $BUCKET \
    --region $REGION \
    --create-bucket-configuration LocationConstraint=$REGION
```

NOTE: us-east-1 does not support a `LocationConstraint`. If your region is `us-east-1`, omit the bucket configuration:

```text
aws s3api create-bucket \
    --bucket $BUCKET \
    --region us-east-1
```

### Create IAM user <a id="create-iam-user"></a>

For more information, see [the AWS documentation on IAM users](http://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html).

1. Create the IAM user:

   ```text
   aws iam create-user --user-name velero
   ```

   If you'll be using Velero to backup multiple clusters with multiple S3 buckets, it may be desirable to create a unique username per cluster rather than the default `velero`.

2. Attach policies to give `velero` the necessary permissions:

   ```text
   cat > velero-policy.json <<EOF
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "ec2:DescribeVolumes",
                   "ec2:DescribeSnapshots",
                   "ec2:CreateTags",
                   "ec2:CreateVolume",
                   "ec2:CreateSnapshot",
                   "ec2:DeleteSnapshot"
               ],
               "Resource": "*"
           },
           {
               "Effect": "Allow",
               "Action": [
                   "s3:GetObject",
                   "s3:DeleteObject",
                   "s3:PutObject",
                   "s3:AbortMultipartUpload",
                   "s3:ListMultipartUploadParts"
               ],
               "Resource": [
                   "arn:aws:s3:::${BUCKET}/*"
               ]
           },
           {
               "Effect": "Allow",
               "Action": [
                   "s3:ListBucket"
               ],
               "Resource": [
                   "arn:aws:s3:::${BUCKET}"
               ]
           }
       ]
   }
   EOF
   ```

   ```text
   aws iam put-user-policy \
     --user-name velero \
     --policy-name velero \
     --policy-document file://velero-policy.json
   ```

3. Create an access key for the user:

   ```text
   aws iam create-access-key --user-name velero
   ```

   The result should look like:

   ```text
   {
     "AccessKey": {
           "UserName": "velero",
           "Status": "Active",
           "CreateDate": "2017-07-31T22:24:41.576Z",
           "SecretAccessKey": <AWS_SECRET_ACCESS_KEY>,
           "AccessKeyId": <AWS_ACCESS_KEY_ID>
     }
   }
   ```

4. Create a Velero-specific credentials file \(`credentials-velero`\) in your local directory:

   ```text
   [default]
   aws_access_key_id=<AWS_ACCESS_KEY_ID>
   aws_secret_access_key=<AWS_SECRET_ACCESS_KEY>
   ```

   where the access key id and secret are the values returned from the `create-access-key` request.

### Install and start Velero

Install Velero, including all prerequisites, into the cluster and start the deployment. This will create a namespace called `velero`, and place a deployment named `velero` in it.

```text
velero install \
    --provider aws \
    --bucket $BUCKET \
    --secret-file ./credentials-velero \
    --backup-location-config region=$REGION \
    --snapshot-location-config region=$REGION
```

Additionally, you can specify `--use-restic` to enable restic support, and `--wait` to wait for the deployment to be ready.

\(Optional\) Specify [additional configurable parameters](https://velero.io/docs/v1.1.0/api-types/backupstoragelocation/#aws) for the `--backup-location-config` flag.

\(Optional\) Specify [additional configurable parameters](https://velero.io/docs/v1.1.0/api-types/volumesnapshotlocation/#aws) for the `--snapshot-location-config` flag.

\(Optional\) Specify [CPU and memory resource requests and limits](https://velero.io/docs/v1.1.0/install-overview/#velero-resource-requirements) for the Velero/restic pods.

For more complex installation needs, use either the Helm chart, or add `--dry-run -o yaml` options for generating the YAML representation for the installation.

### ALTERNATIVE: Setup permissions using kube2iam <a id="alternative-setup-permissions-using-kube2iam"></a>

[Kube2iam](https://github.com/jtblin/kube2iam) is a Kubernetes application that allows managing AWS IAM permissions for pod via annotations rather than operating on API keys.

> This path assumes you have `kube2iam` already running in your Kubernetes cluster. If that is not the case, please install it first, following the docs here: [https://github.com/jtblin/kube2iam](https://github.com/jtblin/kube2iam)

It can be set up for Velero by creating a role that will have required permissions, and later by adding the permissions annotation on the velero deployment to define which role it should use internally.

1. Create a Trust Policy document to allow the role being used for EC2 management & assume kube2iam role:

   ```text
   cat > velero-trust-policy.json <<EOF
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Principal": {
                   "Service": "ec2.amazonaws.com"
               },
               "Action": "sts:AssumeRole"
           },
           {
               "Effect": "Allow",
               "Principal": {
                   "AWS": "arn:aws:iam::<AWS_ACCOUNT_ID>:role/<ROLE_CREATED_WHEN_INITIALIZING_KUBE2IAM>"
               },
               "Action": "sts:AssumeRole"
           }
       ]
   }
   EOF
   ```

2. Create the IAM role:

   ```text
   aws iam create-role --role-name velero --assume-role-policy-document file://./velero-trust-policy.json
   ```

3. Attach policies to give `velero` the necessary permissions:

   ```text
   BUCKET=<YOUR_BUCKET>
   cat > velero-policy.json <<EOF
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "ec2:DescribeVolumes",
                   "ec2:DescribeSnapshots",
                   "ec2:CreateTags",
                   "ec2:CreateVolume",
                   "ec2:CreateSnapshot",
                   "ec2:DeleteSnapshot"
               ],
               "Resource": "*"
           },
           {
               "Effect": "Allow",
               "Action": [
                   "s3:GetObject",
                   "s3:DeleteObject",
                   "s3:PutObject",
                   "s3:AbortMultipartUpload",
                   "s3:ListMultipartUploadParts"
               ],
               "Resource": [
                   "arn:aws:s3:::${BUCKET}/*"
               ]
           },
           {
               "Effect": "Allow",
               "Action": [
                   "s3:ListBucket"
               ],
               "Resource": [
                   "arn:aws:s3:::${BUCKET}"
               ]
           }
       ]
   }
   EOF
   ```

   ```text
   aws iam put-role-policy \
     --role-name velero \
     --policy-name velero-policy \
     --policy-document file://./velero-policy.json
   ```

4. Use the `--pod-annotations` argument on `velero install` to add the following annotation:

```text
velero install \
    --pod-annotations iam.amazonaws.com/role=arn:aws:iam::<AWS_ACCOUNT_ID>:role/<VELERO_ROLE_NAME> \
    --provider aws \
    --bucket $BUCKET \
    --backup-location-config region=$REGION \
    --snapshot-location-config region=$REGION \
    --no-secret
```

