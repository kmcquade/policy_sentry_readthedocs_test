```
Usage: policy_sentry download-policies [OPTIONS]

  Download remote IAM policies to a directory for use in the analyze-iam-
  policies command.

Options:
  --profile TEXT        To authenticate to AWS and analyze *all* existing IAM policies.
  --aws-managed         Use flag if you want to download AWS Managed policies too.
  --include-unattached  Download policies that are unattached too. Defaults to false.
  --help                Show this message and exit.
```

* Make sure you are authenticated to AWS.

### Download customer-managed policies for analysis

* Run this command:

```bash
policy_sentry download-policies --profile dev
```

* It will download the policies to `$HOME/.policy_sentry/policy-analysis/account-number/customer-managed`.

* You can then run analysis on the entire directory:

```bash
policy_sentry analyze-iam-policy --policy $HOME/.policy_sentry/policy-analysis/0123456789012/customer-managed --from-access-level permissions-management
```

Then it will print out the IAM policies that contain actions with "Permissions management" access levels.

### Download AWS Managed policies for analysis

* Run this command:

```bash
policy_sentry download-policies --profile dev --aws-managed
```

* It will download the policies to `$HOME/.policy_sentry/policy-analysis/account-number/aws-managed`.

* You can then run analysis on the entire directory:

```bash
analyze-iam-policy --policy $HOME/.policy_sentry/policy-analysis/0123456789012/customer-managed --from-access-level permissions-management
```

Then it will print out the AWS Managed IAM policies that contain actions with "Permissions management" access levels.

