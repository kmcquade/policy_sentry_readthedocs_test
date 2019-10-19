## Aardvark and Repokid - Policy Revocation

RepoKid is a popular tool that was developed by Netflix, and is one of the more mature and battle-tested AWS IAM open source projects. It leverages AWS Access Advisor, which informs you how many AWS services your IAM Principal has access to, and how many of those services it has used in the last X amount of days or months. If you haven’t used a service within the last 30 days, it “repos” your policy, and strips it of the privileges it doesn’t use. It has some advanced features to allow for whitelisting roles and overall is a great tool. 

One shortcoming is that AWS IAM Access Advisor only  provides details at the service level (ex: S3-wide, or EC2-wide) and not down to the IAM Action level, so the revised policy is not very granular. However, RepoKid plays a unique role in the IAM ecosystem right now in that there are not any open source tools that provide similar functionality. **For that reason, it is best to view RepoKid and Policy Sentry as complimentary.**

Travis McPeak summarized the potential dynamic between Policy Sentry and RepoKid very well on [Clint Gliber’s blog](https://programanalys.is/blog/tldr-sec-010-cloudflare-on-security-iam-least-priv-xss-in-firefox-ui/#policy_sentry---iam-least-privilege-policy-generator):

> Policy Sentry aims to make it easy to create initial least privilege policies and then Repokid takes away unused permissions.

> Creating policies is difficult, so Policy Sentry creates policies based on top level goals and target resources, and then on the backend substitutes the applicable action list to generate the policy. This is very helpful for anybody creating the first version of a policy.

> To help with simplicity these permissions will be assigned somewhat coarsely. So Repokid can use data to remove the specific actions that were granted and aren’t required. Also Repokid will repo down unused permissions once an application stops being used or scope changes.

## AWS-developed tools

### AWS Console - Visual Policy Editor

* [AWS IAM Visual Policy Editor in the AWS Console](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html#access_policies_create-start)

This policy generator is great in general and you should use it if you're getting used to writing IAM policies. 

It's very similar to `policy_sentry` - you are able to bulk select according to access levels. 

However, there are a number of downsides:

* **Missing access level type**: It does not specifically flag "Permissions management" access level
* **No override capabilities for inaccurate Access Levels**: Note how the `ssm:PutParameter` action is listed as "Tagging". This is inaccurate; it should be "Write". Policy_sentry allows you to override inaccurate access levels, whereas the Visual Policy Editor has had inaccurate Access levels for the last several years without any fixes.

> ![Visual Policy Editor - inaccurate access level](images/3-SSM-visual-editor.png)

* **Not automated**: Policy Sentry is, by design, meant for automated policy generation, whereas the Visual Policy Editor is meant to be manual.
* **Console Access**: It also requires access to the AWS Console.
* **Extensibility**: It's open source and Pull Requests are welcome! With `policy_sentry`, we get more control.

On the positive side, it does walk you through creating policies with IAM Condition keys. However, we believe that policy_sentry's approach, where we **always** have policies restricted to the least amount of resources - provides a greater benefit to the end user. Furthermore, we plan on supporting condition keys at some point in the future.

### AWS Policy Generator (static website)

* [AWS Policy Generator - static website](https://awspolicygen.s3.amazonaws.com/policygen.html)

AWS Policy Generator is a great tool; it supports IAM policies, as well as multiple types of resource-based policies (SQS Queue policy, S3 bucket policy, SNS Topic Policy, and VPC Endpoint Policy).

**Loose ARN formatting**: The regex expressions that it uses per-service does not require that actual valid resource ARNs are met - just that they meet the Regex requirement, which is uniform per-service. It just isn't as accurate or up to date as the actual IAM policy generation through the AWS Console

**Missing actions**: To determine the list of actions, it relies on a file titled [policies.js](https://awspolicygen.s3.amazonaws.com/js/policies.js), which contains a list of IAM Actions. However, this file is not as well maintained as the [Actions, Resources, and Condition Keys tables](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_actions-resources-contextkeys.html). For example, it does not have these actions:

```
a4b:describe*
appstream:get*
cloudformation:preview*
codestar:verify*
ds:check*
health:get*
health:list*
kinesisanalytics:get*
lightsail:list*
mobilehub:validate*
resource-groups:describe*
``` 


## Policy Generation from CloudTrail logs

* [CloudTracker](https://github.com/duo-labs/cloudtracker)

Policy Sentry is somewhat similar to CloudTracker. CloudTracker queries CloudTrail logs using Amazon Athena and attempts to “guess” the matching between CloudTrail actions and IAM actions, then generates a policy. Given that there is not a 1-to-1 mapping between the names of Actions listed in CloudTrail log entries and the names AWS IAM Actions, the results are not always accurate. It is a good place to start, but the generated policies all contain Resources: "*", so it is up to the user to restrict those IAM actions to only the necessary resources.

* [Trailscraper](https://github.com/flosell/trailscraper/)

Trailscraper does automated policy generation from CloudTrail logs, but there are some major limitations:
 
1. The generated policies have Resources set to `*``, not to a specific resource ARN 
2. It downloads all of the CloudTrail logs. This takes a while. 
  - Cloudtracker (https://github.com/duo-labs/cloudtracker) uses Amazon Athena, which is more efficient. In the future, I'd like to see a combined approach between all three of these tools to generate IAM policies based on Cloudtrail logs. 3. It is accurate to the point where there is a 1-to-1 mapping with the IAM actions vs CloudTrail logs. As I mentioned in other comments, since not every IAM Action is logged in CloudTrail and not every CloudTrail action matches IAM Actions, the results are not always accurate.


## Infrastructure as Code Tools

* [aws-iam-generator](https://github.com/awslabs/aws-iam-generator)

aws-iam-generator still requires you to write the actual policy templates from scratch, and then they allow you to re-use those policy templates.

Consider the JSON under [this area](https://github.com/awslabs/aws-iam-generator#managed-policies-derived-from-a-jinja2-template) of their README.

It's essentially a method for managing their policies as code - but it doesn't make those policies restricted to certain resources, unless you configure it that way. Using `policy_sentry --write-policy --crud`, you have to supply a file with resource ARNs, and it will write the policy for you, rather than supplying a policy file, and hoping the ARNs fit that use case.


### References

Created some of this FAQ based on questions from [this Hacker News post on policy_sentry](https://news.ycombinator.com/item?id=21262954#21264166).