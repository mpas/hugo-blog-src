+++
title = "Terraform to the rescue"
tags = ["devops", "aws", "terraform"]
date = "2016-10-09"
+++

Getting exposed to [Amazon Web Services](https://aws.amazon.com/) is fun! Certainly when you see that the infrastructure is growing and supporting the daily need of developers and the business. You slowly start adding services and try to keep everything in a state so that it is repeatable and maintainable. At a certain moment it becomes clear that you need the concept of [Infrastructure As Code](https://en.wikipedia.org/wiki/Infrastructure_as_Code).

The Amazon way of doing this is by using [AWS CloudFormation](https://aws.amazon.com/cloudformation/). This enables you to define the infrastructure in a JSON/YAML format and apply the changes to the infrastructure.

Our team manages a bunch of environments using services like AWS ECS, EC2, ElasticSearch, RDS and more.. Maintaining this infrastructure in CloudFormation seemed the standard way of doing things until we started a proof-of-concept with [Terraform](https://www.terraform.io/).

**Why did we start this proof-of-concept??** Mainly because the overwhelming pieces of code that we needed to maintain in CloudFormation became unmaintainable. The use of Terraform was so successful that we decide to rewrite our entire infrastructure codebase using Terraform.

The advantages when using Terraform are:

* less code to maintain because Terraform is less verbose
* when using Terraform an infrastructure change can be planned, this shows what is going to be changed before actually executing the change

```console
$ terraform plan
```
See what the changes are and then when everything seems ok...
```console
$ terraform apply
```

Currently we have our entire Infrastructure in Terraform and we could never be more happier. Terraform came to our rescue!
