
`analyze-iam-policy`: Reads a policy from a JSON file, expands the wildcards (like `s3:List*` if necessary, and audits them to see if certain IAM actions are permitted.

### Options

* `--file`: Supply the requestors IAM policy as a JSON file.
* `--print-policy`: Print the example policy where actions are restricted to ARNs. Defaults to false.
* `--audit-file`: The list of AWS actions to audit. Defaults to `$HOME/.policy_sentry/audit/permissions-access-level.txt`

### Motivation

Let's say you are a developer that handles creation of IAM policies.
* A requestor asks you to create an IAM policy. 
* You haven't been tasked with auditing IAM policies yourself, as that's not your area of expertise, and until this point there is no automation to do it for you.
* However, you want to make sure that the requestors aren't asking for permissions that they don't need, since we need to have *some* guardrails in place to prevent unnecessary exposure of attack surfaces.
* This is made more difficult by the fact that sometimes, the requestor will give you IAM policies that include `*` in the actions. Not only do you want to restrict actions to the specific ARNs, but you want to know what actions they actually need!

You can solve this with policy_sentry too, by auditing for IAM actions in a given policy. Tell them to supply the policy to you in JSON format, and feed it into the `analyze_iam_policy` command, as shown below.

### Instructions

* Build the database:

```bash
policy_sentry initialize
```

#### Audit for permissions with specific access levels

* Command:

```bash
policy_sentry analyze-iam-policy --show permissions-management --file examples/analyze/wildcards.json
```

* Output:

```text
Evaluating: examples/analyze/wildcards.json
Access level: permissions-management
[   'ecr:setrepositorypolicy',
    's3:objectowneroverridetobucketowner',
    's3:putbucketpolicy',
    's3:putbucketacl',
    's3:putobjectacl',
    's3:putobjectversionacl',
    's3:putaccountpublicaccessblock',
    's3:putbucketpublicaccessblock',
    's3:deletebucketpolicy']

```

#### Audit for specific actions

* Command:

```bash
policy_sentry analyze-iam-policy --file examples/analyze/wildcards.json
```

```text
Auditing for risky actions...
Please justify why you need sns:removepermission
Please justify why you need sns:addpermission
Please justify why you need s3:putbucketpublicaccessblock
Please justify why you need s3:objectowneroverridetobucketowner
Please justify why you need s3:putbucketpolicy
Please justify why you need s3:putobjectversionacl
Please justify why you need s3:putaccountpublicaccessblock
Please justify why you need s3:deletebucketpolicy
Please justify why you need s3:putobjectacl
Please justify why you need s3:putbucketacl
Auditing for risky actions complete!
```

* Optionally, you can output the example JSON that shows how to restrict those actions to specific ARNs, by adding the `--print-policy` flag above.

Then you can tell the requestors to create a YAML file to supply to `write_policy --crud`, and create a policy based on that.
