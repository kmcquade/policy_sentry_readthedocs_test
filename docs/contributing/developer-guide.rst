Developer Guide
---------------

Before reading this, make sure you have read all the other documentation - especially the `IAM Policies document <IAM_Policies>`_\ , which covers the Action Tables, ARN tables, and Condition Keys Tables.

Other assumptions:


* You are familiar with these Python things:

  * `click <1>`_
  * package imports, multi folder management
  * PyPi
  * Unit testing

Overall: How policy_sentry uses these tables
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

policy_sentry follows this process for generating policies.


#. If **User-supplied actions** is chosen:

   * Look up the actions in our master Actions Table in the database, which contains the Action Tables for all AWS services
   * If the action in the database matches the actions requested by the user, determine the ARN Format required.
   * Proceed to step 3

#. If **User-supplied ARNs with Access levels** (i.e., the ``--crud`` flag) is chosen: 

   * Match the user-supplied ARNs with ARN formats in our ARN Table database, which contains the ARN tables for all AWS Services
   * If it matches, get the access level requested by the user
   * Proceed to step 3

#. Compile those into groups, sorted by an SID namespace. The SID namespace follows the format of **Service**\ , **Access Level**\ , and **Resource ARN Type**\ , with no character delimiter (to meet AWS IAM Policy formatting expectations). For example, the namespace could be ``SsmReadParameter``\ , ``KmsReadKey``\ , or ``Ec2TagInstance``. 
#. Then, we associate the user-supplied ARNs matching that namespace with the SID.
#. If **User-supplied actions** is chosen:

   * Associate the IAM actions requested by the user to the service, access level, and ARN type matching the aforementioned SID namespace

#. If **User-supplied ARNs with Access levels** (i.e., the ``--crud`` flag) is chosen: 

   * Associate all the IAM actions that correspond to the service, access level, and ARN type matching that SID namespace.

#. Print the policy

Project Structure
^^^^^^^^^^^^^^^^^

We'll focus mostly on the intent and approach of the major files (and subfolders) within the ``policy_sentry/shared`` directory:

Subfolders
~~~~~~~~~~


* ``data/audit/*.txt``\ : These text files are the pre-bundled audit files that you can use with the ``analyze-iam-policy`` command. Currently they are limited to privilege escalation and resource exposure. For more information, see the page on `Analyzing IAM Policies <Analyzing-IAM-Policies>`_.
* ``data/docs/*.html``\ : These are HTML files wget'd from the `Actions, Resources, and Condition Keys <2>`_ AWS documentation. This is used to build our database.
* `data/access-level-overrides.yml`: This is created to override the access levels that AWS incorrectly states in their documentation. For instance, quite often, their service teams will say that an IAM action is "Tagging" when it really should be "Write" - for example, `secretsmanager:CreateSecret`.

Files
~~~~~

**TODO: Insert brief explanations of strategy for some of these later. It will just help people as they try to figure out this project.**

* `actions.py <https://github.com/salesforce/policy_sentry/blob/master/policy_sentry/shared/actions.py>`_
* `analyze.py <https://github.com/salesforce/policy_sentry/blob/master/policy_sentry/shared/analyze.py>`_
* `arns.py <https://github.com/salesforce/policy_sentry/blob/master/policy_sentry/shared/arns.py>`_
* `conditions.py <https://github.com/salesforce/policy_sentry/blob/master/policy_sentry/shared/conditions.py>`_
* `config.py <https://github.com/salesforce/policy_sentry/blob/master/policy_sentry/shared/config.py>`_
* `database.py <https://github.com/salesforce/policy_sentry/blob/master/policy_sentry/shared/database.py>`_
* `download.py <https://github.com/salesforce/policy_sentry/blob/master/policy_sentry/shared/download.py>`_
* `file.py <https://github.com/salesforce/policy_sentry/blob/master/policy_sentry/shared/file.py>`_
* `links.yml <https://github.com/salesforce/policy_sentry/blob/master/policy_sentry/shared/>`_
* `login.py <https://github.com/salesforce/policy_sentry/blob/master/policy_sentry/shared/login.py>`_
* `minimize.py <https://github.com/salesforce/policy_sentry/blob/master/policy_sentry/shared/minimize.py>`_
* `roles.py <https://github.com/salesforce/policy_sentry/blob/master/policy_sentry/shared/roles.py>`_
* `scrape.py <https://github.com/salesforce/policy_sentry/blob/master/policy_sentry/shared/scrape.py>`_
* `template.py <https://github.com/salesforce/policy_sentry/blob/master/policy_sentry/shared/template.py>`_


Pipenv
~~~~~~

.. code-block:: bash

   pipenv --python 3.7  # create the environment
   pipenv shell         # start the environment
   pipenv install       # install both development and production dependencies

Unit Testing
^^^^^^^^^^^^

I use `Nose <https://nose.readthedocs.io/en/latest/>`_ for unit testing. All tests are placed in the ``tests`` folder. 


* Just run the following:

.. code-block:: bash

   nosetests -v

Output:

.. code-block:: text

   Tests the format of the overrides yml file for the RAM service ... ok
   Tests iam:CreateAccessKey (in overrides file as Permissions management, but in the AWS docs as Write) ... ok
   test_get_actions_by_access_level (test_actions.ActionsTestCase) ... ok
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
   test_determine_actions_to_expand: provide expanded list of actions, like ecr:* ... ok
   test_minimize_statement_actions (test_minimize_wildcard_actions.MinimizeWildcardActionsTestCase) ... ok
   test_actions_template (test_template.TemplateTestCase) ... ok
   test_crud_template (test_template.TemplateTestCase) ... ok
   test_print_policy_with_actions_having_dependencies (test_write_policy.WritePolicyActionsTestCase) ... ok
   test_write_policy (test_write_policy.WritePolicyCrudTestCase) ... ok
   test_actions_missing_actions: write-policy actions if the actions block is missing ... ok
   test_allow_missing_access_level_categories_in_cfg: write-policy --crud when the YAML file is missing access level categories ... ok
   test_allow_empty_access_level_categories_in_cfg: If the content of a list is an empty string, it should sysexit ... ok
   test_actions_missing_arn: write-policy actions command when YAML file block is missing an ARN ... ok
   test_actions_missing_description: write-policy when the YAML file is missing a description ... ok
   test_actions_missing_name: write-policy when the YAML file is missing a name? ... ok

Random
^^^^^^


* Code counting: Use `tokei <https://github.com/XAMPPRocky/tokei#how-to-use-tokei>`_

.. code-block:: bash

   tokei ./* --exclude --exclude '**/*.html' --exclude '**/*.json'


Updating the AWS HTML files
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Run the following:

.. code-block:: bash

   ./utils/grab-docs.sh
   # Or:
   ./utils/download-docs.sh