# terraform-aws-named-subnets [![Latest Release](https://img.shields.io/github/release/cloudposse/terraform-aws-named-subnets.svg)](https://github.com/cloudposse/terraform-aws-named-subnets/releases/latest) [![Slack Community](https://slack.cloudposse.com/badge.svg)](https://slack.cloudposse.com)

[![README Header][readme_header_img]][readme_header_link]

[![Cloud Posse][logo]](https://cpco.io/homepage)

<!--




  ** DO NOT EDIT THIS FILE
  **
  ** This file was automatically generated by the `build-harness`.
  ** 1) Make all changes to `README.yaml`
  ** 2) Run `make init` (you only need to do this once)
  ** 3) Run`make readme` to rebuild this file.
  **
  ** (We maintain HUNDREDS of open source projects. This is how we maintain our sanity.)
  **





-->

Terraform module for named [`subnets`](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Subnets.html) provisioning.


---

This project is part of our comprehensive ["SweetOps"](https://cpco.io/sweetops) approach towards DevOps.
[<img align="right" title="Share via Email" src="https://docs.cloudposse.com/images/ionicons/ios-email-outline-2.0.1-16x16-999999.svg"/>][share_email]
[<img align="right" title="Share on Google+" src="https://docs.cloudposse.com/images/ionicons/social-googleplus-outline-2.0.1-16x16-999999.svg" />][share_googleplus]
[<img align="right" title="Share on Facebook" src="https://docs.cloudposse.com/images/ionicons/social-facebook-outline-2.0.1-16x16-999999.svg" />][share_facebook]
[<img align="right" title="Share on Reddit" src="https://docs.cloudposse.com/images/ionicons/social-reddit-outline-2.0.1-16x16-999999.svg" />][share_reddit]
[<img align="right" title="Share on LinkedIn" src="https://docs.cloudposse.com/images/ionicons/social-linkedin-outline-2.0.1-16x16-999999.svg" />][share_linkedin]
[<img align="right" title="Share on Twitter" src="https://docs.cloudposse.com/images/ionicons/social-twitter-outline-2.0.1-16x16-999999.svg" />][share_twitter]


[![Terraform Open Source Modules](https://docs.cloudposse.com/images/terraform-open-source-modules.svg)][terraform_modules]



It's 100% Open Source and licensed under the [APACHE2](LICENSE).







We literally have [*hundreds of terraform modules*][terraform_modules] that are Open Source and well-maintained. Check them out!







## Usage


**IMPORTANT:** The `master` branch is used in `source` just as an example. In your code, do not pin to `master` because there may be breaking changes between releases.
Instead pin to the release tag (e.g. `?ref=tags/x.y.z`) of one of our [latest releases](https://github.com/cloudposse/terraform-aws-named-subnets/releases).



Simple example, with private and public subnets in one Availability Zone:

```hcl
module "vpc" {
  source     = "git::https://github.com/cloudposse/terraform-aws-vpc.git?ref=master"
  namespace  = "eg"
  name       = "vpc"
  stage      = "dev"
  cidr_block = var.cidr_block
}

locals {
  public_cidr_block  = cidrsubnet(module.vpc.vpc_cidr_block, 1, 0)
  private_cidr_block = cidrsubnet(module.vpc.vpc_cidr_block, 1, 1)
}

module "public_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "app"
  subnet_names      = ["web1", "web2", "web3"]
  vpc_id            = module.vpc.vpc_id
  cidr_block        = local.public_cidr_block
  type              = "public"
  igw_id            = module.vpc.igw_id
  availability_zone = "us-east-1a"
}

module "private_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "database"
  subnet_names      = ["kafka", "cassandra", "zookeeper"]
  vpc_id            = module.vpc.vpc_id
  cidr_block        = local.private_cidr_block
  type              = "private"
  availability_zone = "us-east-1a"
  ngw_id            = module.public_subnets.ngw_id
}
```

Simple example, with `ENI` as default route gateway for private subnets

```hcl
resource "aws_network_interface" "default" {
  subnet_id         = module.us_east_1b_public_subnets.subnet_ids[0]
  source_dest_check = false
  tags              = module.network_interface_label.id
}

module "us_east_1b_private_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "app"
  subnet_names      = ["charlie", "echo", "bravo"]
  vpc_id            = module.vpc.vpc_id
  cidr_block        = local.us_east_1b_private_cidr_block
  type              = "private"
  availability_zone = "us-east-1b"
  eni_id            = aws_network_interface.default.id
  attributes        = ["us-east-1b"]
}
```

Full example, with private and public subnets in two Availability Zones for High Availability:

```hcl
module "vpc" {
  source     = "git::https://github.com/cloudposse/terraform-aws-vpc.git?ref=master"
  namespace  = "eg"
  name       = "vpc"
  stage      = "dev"
  cidr_block = var.cidr_block
}

locals {
  us_east_1a_public_cidr_block  = cidrsubnet(module.vpc.vpc_cidr_block, 2, 0)
  us_east_1a_private_cidr_block = cidrsubnet(module.vpc.vpc_cidr_block, 2, 1)
  us_east_1b_public_cidr_block  = cidrsubnet(module.vpc.vpc_cidr_block, 2, 2)
  us_east_1b_private_cidr_block = cidrsubnet(module.vpc.vpc_cidr_block, 2, 3)
}

module "us_east_1a_public_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "app"
  subnet_names      = ["apples", "oranges", "grapes"]
  vpc_id            = module.vpc.vpc_id
  cidr_block        = local.us_east_1a_public_cidr_block
  type              = "public"
  igw_id            = module.vpc.igw_id
  availability_zone = "us-east-1a"
  attributes        = ["us-east-1a"]
}

module "us_east_1a_private_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "app"
  subnet_names      = ["charlie", "echo", "bravo"]
  vpc_id            = module.vpc.vpc_id
  cidr_block        = local.us_east_1a_private_cidr_block
  type              = "private"
  availability_zone = "us-east-1a"
  ngw_id            = module.us_east_1a_public_subnets.ngw_id
  attributes        = ["us-east-1a"]
}

module "us_east_1b_public_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "app"
  subnet_names      = ["apples", "oranges", "grapes"]
  vpc_id            = module.vpc.vpc_id
  cidr_block        = local.us_east_1b_public_cidr_block
  type              = "public"
  igw_id            = module.vpc.igw_id
  availability_zone = "us-east-1b"
  attributes        = ["us-east-1b"]
}

module "us_east_1b_private_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "app"
  subnet_names      = ["charlie", "echo", "bravo"]
  vpc_id            = module.vpc.vpc_id
  cidr_block        = local.us_east_1b_private_cidr_block
  type              = "private"
  availability_zone = "us-east-1b"
  ngw_id            = module.us_east_1b_public_subnets.ngw_id
  attributes        = ["us-east-1b"]
}

resource "aws_network_interface" "default" {
  subnet_id         = module.us_east_1b_public_subnets.subnet_ids[0]
  source_dest_check = false
  tags              = module.network_interface_label.id
}

module "us_east_1b_private_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "app"
  subnet_names      = ["charlie", "echo", "bravo"]
  vpc_id            = module.vpc.vpc_id
  cidr_block        = local.us_east_1b_private_cidr_block
  type              = "private"
  availability_zone = "us-east-1b"
  eni_id            = aws_network_interface.default.id
  attributes        = ["us-east-1b"]
}
```

## Caveat
You must use only one type of device for a default route gateway per route table. `ENI` or `NGW`

Given the following configuration (see the Simple example above)

```hcl
locals {
  public_cidr_block  = cidrsubnet(var.vpc_cidr, 1, 0)
  private_cidr_block = cidrsubnet(var.vpc_cidr, 1, 1)
}

module "public_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "app"
  subnet_names      = ["web1", "web2", "web3"]
  vpc_id            = var.vpc_id
  cidr_block        = local.public_cidr_block
  type              = "public"
  availability_zone = "us-east-1a"
  igw_id            = var.igw_id
}

module "private_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "database"
  subnet_names      = ["kafka", "cassandra", "zookeeper"]
  vpc_id            = var.vpc_id
  cidr_block        = local.private_cidr_block
  type              = "private"
  availability_zone = "us-east-1a"
  ngw_id            = module.public_subnets.ngw_id
}

output "private_named_subnet_ids" {
  value = module.private_subnets.named_subnet_ids
}

output "public_named_subnet_ids" {
  value = module.public_subnets.named_subnet_ids
}
```

the output Maps of subnet names to subnet IDs look like these

```hcl
public_named_subnet_ids = {
  web1 = subnet-ea58d78e
  web2 = subnet-556ee131
  web3 = subnet-6f54db0b
}
private_named_subnet_ids = {
  cassandra = subnet-376de253
  kafka = subnet-9e53dcfa
  zookeeper = subnet-a86fe0cc
}
```

and the created subnet IDs could be found by the subnet names using `map["key"]` or [`lookup(map, key, [default])`](https://www.terraform.io/docs/configuration/interpolation.html#lookup-map-key-default-),

for example:

`public_named_subnet_ids["web1"]`

`lookup(private_named_subnet_ids, "kafka")`






<!-- markdownlint-disable -->
## Makefile Targets
```text
Available targets:

  help                                Help screen
  help/all                            Display help for all targets
  help/short                          This help short screen
  lint                                Lint terraform code

```
<!-- markdownlint-restore -->
## Requirements

| Name | Version |
|------|---------|
| terraform | >= 0.12.0, < 0.14.0 |
| aws | ~> 2.0 |
| null | ~> 2.0 |

## Providers

| Name | Version |
|------|---------|
| aws | ~> 2.0 |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| attributes | Additional attributes (e.g. `policy` or `role`) | `list(string)` | `[]` | no |
| availability\_zone | Availability Zone | `string` | n/a | yes |
| cidr\_block | Base CIDR block which will be divided into subnet CIDR blocks (e.g. `10.0.0.0/16`) | `string` | n/a | yes |
| delimiter | Delimiter to be used between `name`, `namespace`, `stage`, `attributes` | `string` | `"-"` | no |
| enabled | Set to false to prevent the module from creating any resources | `bool` | `true` | no |
| eni\_id | An ID of a network interface which is used as a default route in private route tables (\_e.g.\_ `eni-9c26a123`) | `string` | `""` | no |
| igw\_id | Internet Gateway ID which will be used as a default route in public route tables (e.g. `igw-9c26a123`). Conflicts with `ngw_id` | `string` | `""` | no |
| max\_subnets | Maximum number of subnets which can be created. This variable is being used for CIDR blocks calculation. Defaults to length of `subnet_names` argument | `number` | `16` | no |
| name | Application or solution name | `string` | n/a | yes |
| namespace | Namespace (e.g. `eg` or `cp`) | `string` | `""` | no |
| nat\_enabled | Enable/disable NAT Gateway | `bool` | `true` | no |
| ngw\_id | NAT Gateway ID which will be used as a default route in private route tables (e.g. `igw-9c26a123`). Conflicts with `igw_id` | `string` | `""` | no |
| private\_network\_acl\_egress | Private network egress ACL rules | <pre>list(object(<br>    {<br>      rule_no    = number<br>      action     = string<br>      cidr_block = string<br>      from_port  = number<br>      to_port    = number<br>      protocol   = string<br>  }))</pre> | <pre>[<br>  {<br>    "action": "allow",<br>    "cidr_block": "0.0.0.0/0",<br>    "from_port": 0,<br>    "protocol": "-1",<br>    "rule_no": 100,<br>    "to_port": 0<br>  }<br>]</pre> | no |
| private\_network\_acl\_id | Network ACL ID that will be added to the subnets. If empty, a new ACL will be created | `string` | `""` | no |
| private\_network\_acl\_ingress | Private network ingress ACL rules | <pre>list(object(<br>    {<br>      rule_no    = number<br>      action     = string<br>      cidr_block = string<br>      from_port  = number<br>      to_port    = number<br>      protocol   = string<br>  }))</pre> | <pre>[<br>  {<br>    "action": "allow",<br>    "cidr_block": "0.0.0.0/0",<br>    "from_port": 0,<br>    "protocol": "-1",<br>    "rule_no": 100,<br>    "to_port": 0<br>  }<br>]</pre> | no |
| public\_network\_acl\_egress | Public network egress ACL rules | <pre>list(object(<br>    {<br>      rule_no    = number<br>      action     = string<br>      cidr_block = string<br>      from_port  = number<br>      to_port    = number<br>      protocol   = string<br>  }))</pre> | <pre>[<br>  {<br>    "action": "allow",<br>    "cidr_block": "0.0.0.0/0",<br>    "from_port": 0,<br>    "protocol": "-1",<br>    "rule_no": 100,<br>    "to_port": 0<br>  }<br>]</pre> | no |
| public\_network\_acl\_id | Network ACL ID that will be added to the subnets. If empty, a new ACL will be created | `string` | `""` | no |
| public\_network\_acl\_ingress | Public network ingress ACL rules | <pre>list(object(<br>    {<br>      rule_no    = number<br>      action     = string<br>      cidr_block = string<br>      from_port  = number<br>      to_port    = number<br>      protocol   = string<br>  }))</pre> | <pre>[<br>  {<br>    "action": "allow",<br>    "cidr_block": "0.0.0.0/0",<br>    "from_port": 0,<br>    "protocol": "-1",<br>    "rule_no": 100,<br>    "to_port": 0<br>  }<br>]</pre> | no |
| stage | Stage (e.g. `prod`, `dev`, `staging`) | `string` | `""` | no |
| subnet\_names | List of subnet names (e.g. `['apples', 'oranges', 'grapes']`) | `list(string)` | n/a | yes |
| tags | Additional tags (e.g. map(`BusinessUnit`,`XYZ`) | `map(string)` | `{}` | no |
| type | Type of subnets (`private` or `public`) | `string` | `"private"` | no |
| vpc\_id | VPC ID | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| named\_subnet\_ids | Map of subnet names to subnet IDs |
| ngw\_id | NAT Gateway ID |
| ngw\_private\_ip | Private IP address of the NAT Gateway |
| ngw\_public\_ip | Public IP address of the NAT Gateway |
| route\_table\_ids | Route table IDs |
| subnet\_ids | Subnet IDs |




## Share the Love

Like this project? Please give it a ★ on [our GitHub](https://github.com/cloudposse/terraform-aws-named-subnets)! (it helps us **a lot**)

Are you using this project or any of our other projects? Consider [leaving a testimonial][testimonial]. =)


## Related Projects

Check out these related projects.

- [terraform-aws-multi-az-subnets](https://github.com/cloudposse/terraform-aws-multi-az-subnets) - Terraform module for multi-AZ public and private subnets provisioning.
- [terraform-aws-dynamic-subnets](https://github.com/cloudposse/terraform-aws-dynamic-subnets) - Terraform module for public and private subnets provisioning in existing VPC
- [terraform-aws-vpc](https://github.com/cloudposse/terraform-aws-vpc) - Terraform Module that defines a VPC with public/private subnets across multiple AZs with Internet Gateways
- [terraform-aws-cloudwatch-flow-logs](https://github.com/cloudposse/terraform-aws-cloudwatch-flow-logs) - Terraform module for enabling flow logs for vpc and subnets.



## Help

**Got a question?** We got answers.

File a GitHub [issue](https://github.com/cloudposse/terraform-aws-named-subnets/issues), send us an [email][email] or join our [Slack Community][slack].

[![README Commercial Support][readme_commercial_support_img]][readme_commercial_support_link]

## DevOps Accelerator for Startups


We are a [**DevOps Accelerator**][commercial_support]. We'll help you build your cloud infrastructure from the ground up so you can own it. Then we'll show you how to operate it and stick around for as long as you need us.

[![Learn More](https://img.shields.io/badge/learn%20more-success.svg?style=for-the-badge)][commercial_support]

Work directly with our team of DevOps experts via email, slack, and video conferencing.

We deliver 10x the value for a fraction of the cost of a full-time engineer. Our track record is not even funny. If you want things done right and you need it done FAST, then we're your best bet.

- **Reference Architecture.** You'll get everything you need from the ground up built using 100% infrastructure as code.
- **Release Engineering.** You'll have end-to-end CI/CD with unlimited staging environments.
- **Site Reliability Engineering.** You'll have total visibility into your apps and microservices.
- **Security Baseline.** You'll have built-in governance with accountability and audit logs for all changes.
- **GitOps.** You'll be able to operate your infrastructure via Pull Requests.
- **Training.** You'll receive hands-on training so your team can operate what we build.
- **Questions.** You'll have a direct line of communication between our teams via a Shared Slack channel.
- **Troubleshooting.** You'll get help to triage when things aren't working.
- **Code Reviews.** You'll receive constructive feedback on Pull Requests.
- **Bug Fixes.** We'll rapidly work with you to fix any bugs in our projects.

## Slack Community

Join our [Open Source Community][slack] on Slack. It's **FREE** for everyone! Our "SweetOps" community is where you get to talk with others who share a similar vision for how to rollout and manage infrastructure. This is the best place to talk shop, ask questions, solicit feedback, and work together as a community to build totally *sweet* infrastructure.

## Discourse Forums

Participate in our [Discourse Forums][discourse]. Here you'll find answers to commonly asked questions. Most questions will be related to the enormous number of projects we support on our GitHub. Come here to collaborate on answers, find solutions, and get ideas about the products and services we value. It only takes a minute to get started! Just sign in with SSO using your GitHub account.

## Newsletter

Sign up for [our newsletter][newsletter] that covers everything on our technology radar.  Receive updates on what we're up to on GitHub as well as awesome new projects we discover.

## Office Hours

[Join us every Wednesday via Zoom][office_hours] for our weekly "Lunch & Learn" sessions. It's **FREE** for everyone!

[![zoom](https://img.cloudposse.com/fit-in/200x200/https://cloudposse.com/wp-content/uploads/2019/08/Powered-by-Zoom.png")][office_hours]

## Contributing

### Bug Reports & Feature Requests

Please use the [issue tracker](https://github.com/cloudposse/terraform-aws-named-subnets/issues) to report any bugs or file feature requests.

### Developing

If you are interested in being a contributor and want to get involved in developing this project or [help out](https://cpco.io/help-out) with our other projects, we would love to hear from you! Shoot us an [email][email].

In general, PRs are welcome. We follow the typical "fork-and-pull" Git workflow.

 1. **Fork** the repo on GitHub
 2. **Clone** the project to your own machine
 3. **Commit** changes to your own branch
 4. **Push** your work back up to your fork
 5. Submit a **Pull Request** so that we can review your changes

**NOTE:** Be sure to merge the latest changes from "upstream" before making a pull request!


## Copyright

Copyright © 2017-2020 [Cloud Posse, LLC](https://cpco.io/copyright)



## License

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

See [LICENSE](LICENSE) for full details.

```text
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
```









## Trademarks

All other trademarks referenced herein are the property of their respective owners.

## About

This project is maintained and funded by [Cloud Posse, LLC][website]. Like it? Please let us know by [leaving a testimonial][testimonial]!

[![Cloud Posse][logo]][website]

We're a [DevOps Professional Services][hire] company based in Los Angeles, CA. We ❤️  [Open Source Software][we_love_open_source].

We offer [paid support][commercial_support] on all of our projects.

Check out [our other projects][github], [follow us on twitter][twitter], [apply for a job][jobs], or [hire us][hire] to help with your cloud strategy and implementation.



### Contributors

|  [![Erik Osterman][osterman_avatar]][osterman_homepage]<br/>[Erik Osterman][osterman_homepage] | [![Andriy Knysh][aknysh_avatar]][aknysh_homepage]<br/>[Andriy Knysh][aknysh_homepage] | [![Sergey Vasilyev][s2504s_avatar]][s2504s_homepage]<br/>[Sergey Vasilyev][s2504s_homepage] | [![Vladimir][SweetOps_avatar]][SweetOps_homepage]<br/>[Vladimir][SweetOps_homepage] | [![Konstantin B][comeanother_avatar]][comeanother_homepage]<br/>[Konstantin B][comeanother_homepage] |
|---|---|---|---|---|

  [osterman_homepage]: https://github.com/osterman
  [osterman_avatar]: https://img.cloudposse.com/150x150/https://github.com/osterman.png
  [aknysh_homepage]: https://github.com/aknysh
  [aknysh_avatar]: https://img.cloudposse.com/150x150/https://github.com/aknysh.png
  [s2504s_homepage]: https://github.com/s2504s
  [s2504s_avatar]: https://img.cloudposse.com/150x150/https://github.com/s2504s.png
  [SweetOps_homepage]: https://github.com/SweetOps
  [SweetOps_avatar]: https://img.cloudposse.com/150x150/https://github.com/SweetOps.png
  [comeanother_homepage]: https://github.com/comeanother
  [comeanother_avatar]: https://img.cloudposse.com/150x150/https://github.com/comeanother.png

[![README Footer][readme_footer_img]][readme_footer_link]
[![Beacon][beacon]][website]

  [logo]: https://cloudposse.com/logo-300x69.svg
  [docs]: https://cpco.io/docs?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=docs
  [website]: https://cpco.io/homepage?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=website
  [github]: https://cpco.io/github?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=github
  [jobs]: https://cpco.io/jobs?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=jobs
  [hire]: https://cpco.io/hire?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=hire
  [slack]: https://cpco.io/slack?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=slack
  [linkedin]: https://cpco.io/linkedin?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=linkedin
  [twitter]: https://cpco.io/twitter?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=twitter
  [testimonial]: https://cpco.io/leave-testimonial?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=testimonial
  [office_hours]: https://cloudposse.com/office-hours?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=office_hours
  [newsletter]: https://cpco.io/newsletter?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=newsletter
  [discourse]: https://ask.sweetops.com/?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=discourse
  [email]: https://cpco.io/email?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=email
  [commercial_support]: https://cpco.io/commercial-support?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=commercial_support
  [we_love_open_source]: https://cpco.io/we-love-open-source?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=we_love_open_source
  [terraform_modules]: https://cpco.io/terraform-modules?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=terraform_modules
  [readme_header_img]: https://cloudposse.com/readme/header/img
  [readme_header_link]: https://cloudposse.com/readme/header/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=readme_header_link
  [readme_footer_img]: https://cloudposse.com/readme/footer/img
  [readme_footer_link]: https://cloudposse.com/readme/footer/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=readme_footer_link
  [readme_commercial_support_img]: https://cloudposse.com/readme/commercial-support/img
  [readme_commercial_support_link]: https://cloudposse.com/readme/commercial-support/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-named-subnets&utm_content=readme_commercial_support_link
  [share_twitter]: https://twitter.com/intent/tweet/?text=terraform-aws-named-subnets&url=https://github.com/cloudposse/terraform-aws-named-subnets
  [share_linkedin]: https://www.linkedin.com/shareArticle?mini=true&title=terraform-aws-named-subnets&url=https://github.com/cloudposse/terraform-aws-named-subnets
  [share_reddit]: https://reddit.com/submit/?url=https://github.com/cloudposse/terraform-aws-named-subnets
  [share_facebook]: https://facebook.com/sharer/sharer.php?u=https://github.com/cloudposse/terraform-aws-named-subnets
  [share_googleplus]: https://plus.google.com/share?url=https://github.com/cloudposse/terraform-aws-named-subnets
  [share_email]: mailto:?subject=terraform-aws-named-subnets&body=https://github.com/cloudposse/terraform-aws-named-subnets
  [beacon]: https://ga-beacon.cloudposse.com/UA-76589703-4/cloudposse/terraform-aws-named-subnets?pixel&cs=github&cm=readme&an=terraform-aws-named-subnets
