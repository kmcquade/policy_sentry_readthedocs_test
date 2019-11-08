## Comparison to other tools

* [Comparison to other tools](Comparison-to-other-tools)

## Recommended usage

We recommend using a few measures for locking down your AWS environments with IAM:

1. Use [policy_sentry][0] to create [Identity-based policies][1]
2. Use Service Control Policies (SCPs) to lock down available API calls per account.
  - A great collection of SCPs can be found [here][2].
  - Control Tower has some excellent guidance on strategy for SCPs in their documentation. Note that they call it "Guardrails" but they are mostly SCPs. See the docs [here][3]
3. Use [Repokid][4] to revoke out of date policies as your application/roles mature.
4. Use [Resource-based policies][1] for all services that support them. 
  - A list of which services support resource-based policies can be found [here][5].
5. Never provision infrastructure manually; use Infrastructure as Code 
  - I highly suggest Terraform for IAC over other alternatives such as CloudFormation, Chef, or Puppet. See the Gruntwork blog post [here][6]; Yevgeniy Brikman explains the reasons very well.
  - I also suggest reading HashiCorp's [Unlocking the Cloud Operating Model Whitepaper](https://www.hashicorp.com/cloud-operating-model).

### Other questions

[0]: https://github.com/salesforce/policy_sentry/
[1]: https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_identity-vs-resource.html
[2]: https://asecure.cloud/l/scp/
[3]: https://docs.aws.amazon.com/controltower/latest/userguide/guardrails-reference.html
[4]: https://medium.com/netflix-techblog/introducing-aardvark-and-repokid-53b081bf3a7e
[5]: https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html