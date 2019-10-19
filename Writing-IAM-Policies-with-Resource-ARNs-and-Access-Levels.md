### write_policy: Just specify the CRUD levels

This is the flagship feature of this tool. You can just specify the CRUD levels (Read, Write, List, Tagging, or Permissions management) for each action in a 
YAML File. The policy will be generated for you. You might need to fiddle with the results for your use in Terraform, but it significantly reduces the level of effort to build least privilege into your policies.

## Command options

* `--crud`: Specify this option to use the CRUD functionality. File must be formatted as expected. Defaults to false.
* `--input-file`: YAML file containing the CRUD levels + Resource ARNs. Required.
* `--minimize`: Whether or not to minimize the resulting statement with *safe* usage of wildcards to reduce policy length. Set this to the character length you want. This can be extended for readability. I suggest setting it to `0`.

Example:

```bash
policy_sentry write-policy --crud --input-file examples/crud.yml
```

## Instructions

* Create a YAML file with the following contents:

```yaml
roles_with_crud_levels:
- name: 'RoleNameWithCRUD'
  description: 'Why I need these privs'
  arn: 'arn:aws:iam::123456789012:role/RiskyEC2'
  read:
    - arn:aws:s3:::example-org-sbx-vmimport
    - arn:aws:s3:::example-kinnaird
    - arn:aws:ssm:us-east-1:123456789012:parameter/test
    - arn:aws:ssm:us-east-1:123456789012:parameter/test2
    - arn:aws:kms:us-east-1:123456789012:key/123456
    - arn:aws:s3:::job/jobid
    - arn:aws:s3:::example-org-sbx-vmimport/stuff
  write:
    - arn:aws:s3:::example-org-s3-access-logs
    - arn:aws:s3:::example-org-sbx-vmimport/stuff
    - arn:aws:secretsmanager:us-east-1:123456789012:secret:mysecret
    - arn:aws:kms:us-east-1:123456789012:key/123456
  list:
    - arn:aws:s3:::example-org-flow-logs
    - arn:aws:s3:::example-org-sbx-vmimport/stuff
  tag:
    - arn:aws:ssm:us-east-1:123456789012:parameter/test
  permissions-management:
    - arn:aws:s3:::example-org-s3-access-logs

```

* Run the command:

```bash
policy_sentry write-policy --crud --input-file examples/crud.yml
```

* It will generate an IAM Policy containing an IAM policy with the actions restricted to the ARNs specified above.
* The resulting policy (without the `--minimize command`) will look like this:


```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "S3ReadBucket",
            "Effect": "Allow",
            "Action": [
                "s3:getaccelerateconfiguration",
                "s3:getanalyticsconfiguration",
                "s3:getbucketacl",
                "s3:getbucketcors",
                "s3:getbucketlocation",
                "s3:getbucketlogging",
                "s3:getbucketnotification",
                "s3:getbucketpolicy",
                "s3:getbucketpolicystatus",
                "s3:getbucketpublicaccessblock",
                "s3:getbucketrequestpayment",
                "s3:getbuckettagging",
                "s3:getbucketversioning",
                "s3:getbucketwebsite",
                "s3:getencryptionconfiguration",
                "s3:getinventoryconfiguration",
                "s3:getlifecycleconfiguration",
                "s3:getmetricsconfiguration",
                "s3:getreplicationconfiguration",
                "s3:listbucketbytags",
                "s3:listbucketmultipartuploads",
                "s3:listbucketversions"
            ],
            "Resource": [
                "arn:aws:s3:::example-org-sbx-vmimport",
                "arn:aws:s3:::example-kinnaird"
            ]
        },
        {
            "Sid": "SsmReadParameter",
            "Effect": "Allow",
            "Action": [
                "ssm:getparameter",
                "ssm:getparameterhistory",
                "ssm:getparameters",
                "ssm:getparametersbypath",
                "ssm:listtagsforresource"
            ],
            "Resource": [
                "arn:aws:ssm:us-east-1:123456789012:parameter/test",
                "arn:aws:ssm:us-east-1:123456789012:parameter/test2"
            ]
        },
        {
            "Sid": "KmsReadKey",
            "Effect": "Allow",
            "Action": [
                "kms:describekey",
                "kms:getkeypolicy",
                "kms:getkeyrotationstatus",
                "kms:getparametersforimport",
                "kms:listresourcetags"
            ],
            "Resource": [
                "arn:aws:kms:us-east-1:123456789012:key/123456"
            ]
        },
        {
            "Sid": "S3ReadJob",
            "Effect": "Allow",
            "Action": [
                "s3:describejob"
            ],
            "Resource": [
                "arn:aws:s3:::job/jobid"
            ]
        },
        {
            "Sid": "S3ReadObject",
            "Effect": "Allow",
            "Action": [
                "s3:getobject",
                "s3:getobjectacl",
                "s3:getobjecttagging",
                "s3:getobjecttorrent",
                "s3:getobjectversion",
                "s3:getobjectversionacl",
                "s3:getobjectversionforreplication",
                "s3:getobjectversiontagging",
                "s3:getobjectversiontorrent",
                "s3:listmultipartuploadparts"
            ],
            "Resource": [
                "arn:aws:s3:::job/jobid",
                "arn:aws:s3:::example-org-sbx-vmimport/stuff"
            ]
        },
        {
            "Sid": "S3WriteBucket",
            "Effect": "Allow",
            "Action": [
                "s3:createbucket",
                "s3:deletebucket",
                "s3:deletebucketwebsite",
                "s3:getbucketobjectlockconfiguration",
                "s3:putaccelerateconfiguration",
                "s3:putanalyticsconfiguration",
                "s3:putbucketcors",
                "s3:putbucketlogging",
                "s3:putbucketnotification",
                "s3:putbucketobjectlockconfiguration",
                "s3:putbucketrequestpayment",
                "s3:putbucketversioning",
                "s3:putbucketwebsite",
                "s3:putencryptionconfiguration",
                "s3:putinventoryconfiguration",
                "s3:putlifecycleconfiguration",
                "s3:putmetricsconfiguration",
                "s3:putreplicationconfiguration"
            ],
            "Resource": [
                "arn:aws:s3:::example-org-s3-access-logs"
            ]
        },
        {
            "Sid": "S3WriteObject",
            "Effect": "Allow",
            "Action": [
                "s3:abortmultipartupload",
                "s3:deleteobject",
                "s3:deleteobjectversion",
                "s3:getobjectlegalhold",
                "s3:getobjectretention",
                "s3:putobject",
                "s3:putobjectlegalhold",
                "s3:putobjectretention",
                "s3:replicatedelete",
                "s3:replicateobject",
                "s3:restoreobject"
            ],
            "Resource": [
                "arn:aws:s3:::example-org-sbx-vmimport/stuff"
            ]
        },
        {
            "Sid": "SecretsmanagerWriteSecret",
            "Effect": "Allow",
            "Action": [
                "secretsmanager:cancelrotatesecret",
                "secretsmanager:deletesecret",
                "secretsmanager:putsecretvalue",
                "secretsmanager:restoresecret",
                "secretsmanager:rotatesecret",
                "secretsmanager:updatesecret",
                "secretsmanager:updatesecretversionstage"
            ],
            "Resource": [
                "arn:aws:secretsmanager:us-east-1:123456789012:secret:mysecret"
            ]
        },
        {
            "Sid": "KmsWriteKey",
            "Effect": "Allow",
            "Action": [
                "kms:cancelkeydeletion",
                "kms:createalias",
                "kms:decrypt",
                "kms:deletealias",
                "kms:deleteimportedkeymaterial",
                "kms:disablekey",
                "kms:disablekeyrotation",
                "kms:enablekey",
                "kms:enablekeyrotation",
                "kms:encrypt",
                "kms:generatedatakey",
                "kms:generatedatakeywithoutplaintext",
                "kms:importkeymaterial",
                "kms:reencryptfrom",
                "kms:reencryptto",
                "kms:schedulekeydeletion",
                "kms:updatealias",
                "kms:updatekeydescription"
            ],
            "Resource": [
                "arn:aws:kms:us-east-1:123456789012:key/123456"
            ]
        },
        {
            "Sid": "S3ListBucket",
            "Effect": "Allow",
            "Action": [
                "s3:listbucket"
            ],
            "Resource": [
                "arn:aws:s3:::example-org-flow-logs"
            ]
        },
        {
            "Sid": "S3PermissionsmanagementBucket",
            "Effect": "Allow",
            "Action": [
                "s3:deletebucketpolicy",
                "s3:putbucketacl",
                "s3:putbucketpolicy",
                "s3:putbucketpublicaccessblock"
            ],
            "Resource": [
                "arn:aws:s3:::example-org-s3-access-logs"
            ]
        },
        {
            "Sid": "SsmTaggingParameter",
            "Effect": "Allow",
            "Action": [
                "ssm:addtagstoresource",
                "ssm:putparameter",
                "ssm:removetagsfromresource"
            ],
            "Resource": [
                "arn:aws:ssm:us-east-1:123456789012:parameter/test"
            ]
        }
    ]
}

```

