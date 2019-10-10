## Developer Guide

Before reading this, make sure you have read all the other documentation - especially the [IAM Policies document](IAM_Policies), which covers the Action Tables, ARN tables, and Condition Keys Tables.

### Overall: How policy_sentry uses these tables

policy_sentry follows this process for generating policies.

1. If __User-supplied actions__ is chosen:
  - Look up the actions in our master Actions Table in the database, which contains the Action Tables for all AWS services
  - If the action in the database matches the actions requested by the user, determine the ARN Format required.
  - Proceed to step 3
2. If __User-supplied ARNs with Access levels__ (i.e., the `--crud` flag) is chosen: 
  - Match the user-supplied ARNs with ARN formats in our ARN Table database, which contains the ARN tables for all AWS Services
  - If it matches, get the access level requested by the user
  - Proceed to step 3
3. Compile those into groups, sorted by an SID namespace. The SID namespace follows the format of **Service**, **Access Level**, and **Resource ARN Type**, with no character delimiter (to meet AWS IAM Policy formatting expectations). For example, the namespace could be `SsmReadParameter`, `KmsReadKey`, or `Ec2TagInstance`. 
4. Then, we associate the user-supplied ARNs matching that namespace with the SID.
5. If __User-supplied actions__ is chosen:
  - Associate the IAM actions requested by the user to the service, access level, and ARN type matching the aforementioned SID namespace
6. If __User-supplied ARNs with Access levels__ (i.e., the `--crud` flag) is chosen: 
  - Associate all the IAM actions that correspond to the service, access level, and ARN type matching that SID namespace.
7. Print the policy

### Project Structure

Fill this in later.

## Development

Install dependencies using pipenv.

#### Pipenv

```bash
pipenv --python 3.7  # create the environment
pipenv shell         # start the environment
pipenv install       # install both development and production dependencies
```

### Unit Testing

I use [Nose](https://nose.readthedocs.io/en/latest/) for unit testing. All tests are placed in the `tests` folder. 

* Just run the following:

```bash
nosetests -v
```

Output:

```text
test_get_dependent_actions_double (test_actions.ActionsTestCase) ... ok
test_get_dependent_actions_several (test_actions.ActionsTestCase) ... ok
test_get_dependent_actions_single (test_actions.ActionsTestCase) ... ok
test_add_s3_permissions_management_arn (test_arn_action_group.ArnActionGroupTestCase) ... ok
test_get_policy_elements (test_arn_action_group.ArnActionGroupTestCase) ... ok
test_update_actions_for_raw_arn_format (test_arn_action_group.ArnActionGroupTestCase) ... ok
test_does_arn_match_case_1 (test_arns.ArnsTestCase) ... ok
test_does_arn_match_case_2 (test_arns.ArnsTestCase) ... ok
test_does_arn_match_case_4 (test_arns.ArnsTestCase) ... ok
test_does_arn_match_case_5 (test_arns.ArnsTestCase) ... ok
test_does_arn_match_case_6 (test_arns.ArnsTestCase) ... ok
test_does_arn_match_case_bucket (test_arns.ArnsTestCase) ... ok
test_print_policy_with_actions_having_dependencies (test_write_policy.WritePolicyActionsTestCase) ... ok
test_write_policy (test_write_policy.WritePolicyCrudTestCase) ... ok

----------------------------------------------------------------------
Ran 14 tests in 1.138s

OK
```
