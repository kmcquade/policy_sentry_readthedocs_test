Supply a list of actions in a YAML file and generate the policy accordingly.

## Command options

* `--file`: YAML file containing the list of actions
* `--minimize`: Whether or not to minimize the resulting statement with *safe* usage of wildcards to reduce policy length. Set this to the character lengh you want - for example, 4

Example:
```bash
policy_sentry write-policy --file examples/actions.yml
```

## Instructions

* Create a yaml file with the following contents:

```yaml
roles_with_actions:
- name: 'RoleNameWithActions'
  description: 'Why I need these privs'
  arn: 'arn:aws:iam::123456789102:role/RiskyEC2'
  actions:
  - kms:CreateGrant
  - kms:CreateCustomKeyStore
  - ec2:AuthorizeSecurityGroupEgress
  - ec2:AuthorizeSecurityGroupIngress
```

* Then run this in command line:

```bash
policy_sentry write-policy --file examples/actions.yml
```

* The output will look like this:

```text

=======
RoleNameWithActions
-------
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "LeastPriv0",
            "Effect": "Allow",
            "Action": [
                "kms:CreateCustomKeyStore",
                "ec2:DescribeInstances",
                "cloudhsm:DescribeClusters"
            ],
            "Resource": "*"
        },
        {
            "Sid": "LeastPriv1",
            "Effect": "Allow",
            "Action": [
                "kms:CreateGrant"
            ],
            "Resource": "arn:aws:kms:${Region}:${Account}:key/${KeyId}"
        }
    ]
}
=======
SecondRoleNameWithActions
-------
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "LeastPriv0",
            "Effect": "Allow",
            "Action": [
                "ec2:AuthorizeSecurityGroupEgress",
                "ec2:AuthorizeSecurityGroupIngress"
            ],
            "Resource": "arn:aws:ec2:${Region}:${Account}:security-group/${SecurityGroupId}"
        }
    ]
}

```
