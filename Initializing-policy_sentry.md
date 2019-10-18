`initialize`: This will create a SQLite database that contains all of the services available through the [Actions, Resources, and Condition Keys documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_actions-resources-contextkeys.html).

The database is stored in `$HOME/.policy_sentry/aws.sqlite3`.

The database is generated based on the HTML files stored in the `policy_sentry/shared/data/docs/` directory.

### Options

* `--access-level-overrides-file`: Path to your own custom access level overrides file, used to override the Access Levels per action provided by AWS docs. The default one is [here](https://github.com/salesforce/policy_sentry/blob/master/policy_sentry/shared/data/access-level-overrides.yml).

### Usage

```bash
# Initialize the database, using the existing Access Level Overrides file
policy_sentry initialize

# Initialize the database with a custom Access Level Overrides file
policy_sentry initialize --access-level-overrides-file my-override.yml
```
