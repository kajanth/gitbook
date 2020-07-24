# terraform scan

## Getting Super Powers

Becoming a super hero is a fairly straight forward process:

```
terraform fmt -check
terraform validate

wget https://github.com/wata727/tflint/releases/download/v0.5.4/tflint_linux_amd64.zip -O /tmp/tflint.zip
sudo unzip /tmp/tflint.zip -d /usr/local/bin

tflint -h

tflint

tflint --error-with-issues
echo $?

#Print out the exit code again but include the additional --ignore-rule option:
tflint --error-with-issues --ignore-rule=aws_instance_not_specified_iam_profile
echo $?

#Attempt to run a TFLint deep check:
tflint --deep --aws-region=us-west-2

#Run the normal scan that ignores the notice rule, but add the debug option to list the rules that are in TFLint:
tflint --error-with-issues --ignore-rule=aws_instance_not_specified_iam_profile --debug

terrascan

cd
virtualenv scan
source scan/bin/activate
pip install -Iv terrascan==0.1.0
cd environment/tf

terrascan --location . --tests security_group

deactivate

aws ec2 describe-instances \
--filters "Name=tag:Type,Values=Build" \
--query "Reservations[0].Instances[0].PublicDnsName" \

sed 's/"(.*)"/http:\/\/\1\/manage/'
#!/bin/bash
docker run -v $(pwd):/src --workdir=/src --rm wata727/tflint:0.5.4 --error-with-issues

This 

cd ~/environment
repo_url=$(aws ec2 describe-instances --filters "Name=tag:Type,Values=Build" --query "Reservations[0].Instances[0].PublicDnsName" \

sed 's/"(.*)"/git:\/\/\1\/lab.git/')
git clone $repo_url src
cp tf/*tf tf/*sh src
cd src
git add -A
git commit -m "Initial commit”
```

