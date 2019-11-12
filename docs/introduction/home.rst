Overview
=============

Policy Sentry is an IAM Least Privilege Policy Generator, auditor, and analysis database.

Motivation
----------

Writing security-conscious IAM Policies by hand can be very tedious and inefficient. Many Infrastructure as Code developers have experienced something like this:


* Determined to make your best effort to give users and roles the least amount of privilege you need to perform your duties, you spend way too much time combing through the AWS IAM Documentation on `Actions, Resources, and Condition Keys for AWS Services <https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_actions-resources-contextkeys.html>`_.
* Your team lead encourages you to build security into your IAM Policies for product quality, but eventually you get frustrated due to project deadlines.
* You don't have an embedded security person on your team who can write those IAM policies for you, and there's no automated tool that will automagically sense the AWS API calls that you perform and then write them for you in a least-privilege manner.
* After fantasizing about that level of automation, you realize that writing least privilege IAM Policies, seemingly out of charity, will jeopardize your ability to finish your code in time to meet project deadlines.
* You use Managed Policies (because hey, why not) or you eyeball the names of the API calls and use wildcards instead so you can move on with your life.

Such a process is not ideal for security or for Infrastructure as Code developers. We need to make it easier to write IAM Policies securely and abstract the complexity of writing least-privilege IAM policies. That's why I made this tool.

Authoring Secure IAM Policies
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``policy_sentry``\ 's flagship feature is that it can create IAM policies based on resource ARNs and access levels. Our CRUD functionality takes the opinionated approach that IAC developers shouldn't have to understand the complexities of AWS IAM - we should abstract the complexity for them. In fact, developers should just be able to say...


* "I need Read/Write/List access to ``arn:aws:s3:::example-org-sbx-vmimport``\ "
* "I need Permissions Management access to ``arn:aws:secretsmanager:us-east-1:123456789012:secret:mysecret``\ "
* "I need Tagging access to ``arn:aws:ssm:us-east-1:123456789012:parameter/test``\ "

...and our automation should create policies that correspond to those access levels.


How do we accomplish this? Well, policy_sentry leverages the AWS documentation on `Actions, Resources, and Condition Keys <1>`_ documentation to look up the actions, access levels, and resource types, and generates policies according to the ARNs and access levels. Consider the table snippet below:

+----------------------------------+------------------------+--------------------+
| **Actions**                      | **Access Level**       | **Resource Types** |
+----------------------------------+------------------------+--------------------+
| ssm:GetParameter                 | Read                   | parameter          |
+----------------------------------+------------------------+--------------------+
| ssm:DescribeParameters           | List                   | parameter          |
+----------------------------------+------------------------+--------------------+
| ssm:PutParameter                 | Write                  | parameter          |
+----------------------------------+------------------------+--------------------+
| secretsmanager:PutResourcePolicy | Permissions management | secret             |
+----------------------------------+------------------------+--------------------+
| secretsmanager:TagResource       | Tagging                | secret             |
+----------------------------------+------------------------+--------------------+

Policy Sentry aggregates all of that documentation into a single database and uses that database to generate policies according to actions, resources, and access levels. To generate a policy according to resources and access levels, start by creating a template with this command so you can just fill out the ARNs:

.. code-block:: bash

   policy_sentry create-template --name myRole --output-file crud.yml --template-type crud

It will generate a file like this:

.. code-block:: yaml

   roles_with_crud_levels:
   - name: myRole
     description: '' # Insert description
     arn: '' # Insert the ARN of the role that will use this
     read:
       - '' # Insert ARNs for Read access
     write:
       - '' # Insert ARNs...
     list:
       - '' # Insert ARNs...
     tag:
       - '' # Insert ARNs...
     permissions-management:
       - '' # Insert ARNs...

Then just fill it out:

.. code-block:: yaml

   roles_with_crud_levels:
   - name: myRole
     description: 'Justification for privileges'
     arn: 'arn:aws:iam::123456789102:role/myRole'
     read:
       - 'arn:aws:ssm:us-east-1:123456789012:parameter/myparameter'
     write:
       - 'arn:aws:ssm:us-east-1:123456789012:parameter/myparameter'
     list:
       - 'arn:aws:ssm:us-east-1:123456789012:parameter/myparameter'
     tag:
       - 'arn:aws:secretsmanager:us-east-1:123456789012:secret:mysecret'
     permissions-management:
       - 'arn:aws:secretsmanager:us-east-1:123456789012:secret:mysecret'

Then run this command:

.. code-block:: bash

   policy_sentry write-policy --crud --input-file crud.yml

It will generate these results:

.. code-block:: json

   {
       "Version": "2012-10-17",
       "Statement": [
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
                   "arn:aws:ssm:us-east-1:123456789012:parameter/myparameter"
               ]
           },
           {
               "Sid": "SsmWriteParameter",
               "Effect": "Allow",
               "Action": [
                   "ssm:deleteparameter",
                   "ssm:deleteparameters",
                   "ssm:putparameter",
                   "ssm:labelparameterversion"
               ],
               "Resource": [
                   "arn:aws:ssm:us-east-1:123456789012:parameter/myparameter"
               ]
           },
           {
               "Sid": "SecretsmanagerPermissionsmanagementSecret",
               "Effect": "Allow",
               "Action": [
                   "secretsmanager:deleteresourcepolicy",
                   "secretsmanager:putresourcepolicy"
               ],
               "Resource": [
                   "arn:aws:secretsmanager:us-east-1:123456789012:secret:mysecret"
               ]
           },
           {
               "Sid": "SecretsmanagerTaggingSecret",
               "Effect": "Allow",
               "Action": [
                   "secretsmanager:tagresource",
                   "secretsmanager:untagresource"
               ],
               "Resource": [
                   "arn:aws:secretsmanager:us-east-1:123456789012:secret:mysecret"
               ]
           }
       ]
   }

Notice how the policy above recognizes the ARNs that the user supplies, along with the requested access level. For instance, the SID “SecretsmanagerTaggingSecret” contains Tagging actions that are assigned to the secret resource type only.

This rapidly speeds up the time to develop IAM policies, and ensures that all policies created limit access to exactly what your role needs access to. This way, developers only have to determine the resources that they need to access, and we abstract the complexity of IAM policies away from their development processes.

Installation
-------------


* ``policy_sentry`` is available via pip. To install, run:

.. code-block:: bash

   pip install --user policy_sentry



Usage
^^^^^


*
  ``initialize``\ : Create a SQLite database that contains all of the services available through the `Actions, Resources, and Condition Keys documentation <https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_actions-resources-contextkeys.html>`_. See the `documentation <https://github.com/salesforce/policy_sentry/wiki/Initializing-policy_sentry>`_.

*
  ``create-template``\ : Creates the YML file templates for use in the ``write-policy`` command types.

*
  ``write-policy``\ : Leverage a YAML file to write policies for you


  * Option 1: Specify CRUD levels (Read, Write, List, Tagging, or Permissions management) and the ARN of the resource. It will write this for you. See the `documentation on CRUD mode <https://github.com/salesforce/policy_sentry/wiki/Writing-IAM-Policies-with-Resource-ARNs-and-Access-Levels>`_
  * Option 2: Specify a list of actions. It will write the IAM Policy for you, but you will have to fill in the ARNs. See the `documentation on Action Mode <https://github.com/salesforce/policy_sentry/wiki/Writing-IAM-Policies-with-a-List-of-Actions>`_.

*
  ``write-policy-dir``\ : This can be helpful in the Terraform use case. For more information, see the wiki.

*
  ``download-policies``\ : Download IAM policies from your AWS account for analysis.

*
  `analyze-iam-policy`: Analyze an IAM policy read from a JSON file, expands the wildcards (like `s3:List*` if necessary.


  * Option 1: Audits them to see if certain IAM actions are permitted, based on actions in a separate text file. See the `documentation on Initialization <https://github.com/salesforce/policy_sentry/wiki/Initializing-policy_sentry>`_.
  * Option 2: Audits them to see if any of the actions in the policy meet a certain access level, such as "Permissions management."


Author Information
------------------

Author:

* `Kinnaird McQuade <https://github.com/kmcquade>`_

  * `Twitter <https://twitter.com/kmcquade3>`__
  * `Keybase <https://keybase.io/kmcquade>`__
  * `LinkedIn <https://www.linkedin.com/in/kinnairdmcquade/>`__

Contributors:

* `Matt Jones <https://github.com/mattyjones>`__

  * `Twitter <https://twitter.com/CaffeinatedEng>`__
  * `Keybase <https://keybase.io/urlugal>`__
  * `LinkedIn <https://www.linkedin.com/in/mattyjones/>`__
