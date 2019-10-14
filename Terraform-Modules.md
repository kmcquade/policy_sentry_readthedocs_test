## Using the Terraform modules

### Step 0: Install policy_sentry

* Install policy_sentry

```bash
pip3 install --user git+https://github.com/salesforce/policy_sentry.git#egg=policy_sentry
```

* Initialize policy_sentry

```bash
policy_sentry initialize
```


### Step 1: Generate the policy_sentry YAML File
Create a file with the following in `some-directory`:

```hcl-terraform
module "policy_sentry_yml" {
  source           = "git::https://github.com/salesforce/policy_sentry.git//examples/terraform/modules/generate-policy_sentry-yml"
  role_name        = ""
  role_description = ""
  role_arn         = ""

  list_access_level                   = []
  permissions_management_access_level = []
  read_access_level                   = []
  tagging_access_level                = []
  write_access_level                  = []

  yml_file_destination_path = "../other-directory/files"
}
```

Make sure you fill out the actual directory path properly. Note that `yml_file_destination_path` should point to the directory mentioned in Step 3.

### Step 2: Run policy_sentry and specify proper target directory

* Enter the directory you specified under `yml_file_destination_path` above.

* Run the following:

```bash
policy_sentry write-policy-dir --crud --input-dir files --output-dir files
```

### Step 3: Create the IAM Policies using JSON files from directory

Then from `other-directory`:

```hcl-terraform
module "policies" {
  source = "git::https://github.com/salesforce/policy_sentry.git//examples/terraform/modules/generate-iam-policies"
  relative_path_to_json_policy_files = "files"
}
```