`initialize`: This will create a SQLite database that contains all of the services available through the [Actions, Resources, and Condition Keys documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_actions-resources-contextkeys.html).

The database is stored in `$HOME/.policy_sentry/aws.sqlite3`.

The database is generated based on the HTML files stored in the `policy_sentry/shared/data/docs/` directory.

### Options

None

### Usage

```bash
policy_sentry initialize
```
