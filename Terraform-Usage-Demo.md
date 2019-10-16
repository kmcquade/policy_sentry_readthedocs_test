Please download the demo code [here](https://github.com/salesforce/policy_sentry/tree/master/examples/terraform) to follow along.

## Prerequisites

This requires:
* Terraform v0.12.8
* AWS credentials; must be authenticated

### Commands

* Install policy_sentry

```bash
pip3 install policy_sentry
```

* Initialize policy_sentry

```bash
policy_sentry initialize
```

* Execute the first Terraform module:

```bash
cd environments/standard-resources
tfjson install 0.12.8
terraform init
terraform plan
terraform apply -auto-approve
```

This will create a YML file to be used by policy_sentry in the [environments/iam-resources/files/](https://github.com/salesforce/policy_sentry/tree/master/examples/terraform/environments/iam-resources/files) directory titled [example-role-randomid.yml](https://github.com/salesforce/policy_sentry/blob/master/examples/terraform/environments/iam-resources/files/example-role-jpwdp.yml.example).

* Write the policy using policy_sentry:

```bash
cd ../iam-resources
policy_sentry write-policy-dir --crud --input-dir files --output-dir files
```

This will create a JSON file to be consumed by Terraform's `aws_iam_policy` resource to create an IAM policy.

* Now create the policies with Terraform:

```bash
terraform init
terraform plan
terraform apply -auto-approve
```

* Don't forget to cleanup

```bash
terraform destroy -auto-approve
cd ../standard-resources
terraform destroy -auto-approve
```
