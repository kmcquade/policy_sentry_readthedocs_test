# policy_sentry

policy_sentry is an IAM Least Privilege Policy Generator, auditor, and analysis database.

Please note that the wiki documentation is a work in progress.

## Navigating the Wiki

### Pre-requisite knowledge
  * [IAM Policies](IAM_Policies). This covers (1) Elements of an IAM Policy and (2) Breakdown of the tables for Actions, Resources, and Condition keys per service
  * [Minimization](Minimization). This covers our method for reducing the size of IAM policies generated using our tool in an attempt to meet IAM Policy Character limit restrictions.

### Getting Started as a User
  * [User Guide](User-Guide)
  * [Initialization](Initializing-policy_sentry): How to initialize policy_sentry. This creates the SQLite database stored in the `$HOME/.policy_sentry/` directory, which is required for using this tool.
  * [Write Policies with Resource ARNs and Access Levels](Writing-IAM-Policies-with-Resource-ARNs-and-Access-Levels)
  * [Write Policies with a List of Actions](Writing-IAM-Policies-with-a-List-of-Actions)
  * [Analyzing IAM Policies](Analyzing-IAM-Policies)

### Creating Policies with Terraform and policy_sentry
  * [Demo: Terraform and policy_sentry](Terraform-Usage-Demo): This provides a walkthrough of the Terraform + policy_sentry demo code. 
  * [Terraform Modules](Terraform-Modules): How to generate policies with policy_sentry + Terraform using pre-built Terraform modules.

### Getting Started as a Developer
* [Developer Guide](Developer-Guide). This covers generally how policy_sentry uses the Action Tables, ARN Tables, and Condition Keys tables to generate IAM Policies

### Other
* [Limitations](Limitations): Identifies some services that are currently missing
