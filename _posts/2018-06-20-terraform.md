---
layout: post
title:  "Shared Terraform State file"
date:   2018-11-20
desc: "How to manage AWS state file in a shared environment"
keywords: "Terraform, AWS, S3, shared, remote, backend "
categories: [Aws]
tags: [terraform,AWS,S3,Backend]
icon: icon-html
---

Probably you are not familiarized with the infrastucture state concept, let's say that it's a snapshot about what do we have configured and running in our current infrastucture **(VPC's, Subnets, Instances, RDS, Volumes, etc)**, this snapshot help us to update our infrastructure allowing us to run our scripts and applying just the changes that we have, this snapshot is what provide us idempotence in IT architecture creation.

Now, let's say that we have a git repo with our terraform scripts, those scripts creates a VPC, two subnets and two instances one inside each subnet, this git repo has collaborators that can execute the deploy when they want; but, what happen with that **state file**?; that state file cannot be stored in git because is a file that has sensitive info about our architecture and it changes quite often.

So, we have an issue here, what would happen if two collaborators apply their changes without share their **state file**?. We will have infrastructure consistency errors because some changes will be executed before other and you could delete the changes made by another collaborator.

then, What can we do?

In this scenario, Terraform provides us a remote state feature called [**backends**](https://www.terraform.io/docs/backends/), that allow us to have an updated state file with every iteration or execution.

Now what would happen if you want to test your changes without change the develop or QA or even production environment? At this escenario we could use a simple Terraform tool called **workspaces**, this workspaces will allow us to create different environments that will have their own state file.

This workspace will not block the write over a state file, just will give us the chance to work with different state files and different stacks.

To work with this configuration follow the next steps:

1. Create an account with S3 access

2. Create bucket for save terraform.tfstate files

```
YOUR-BUCKET-NAME
```
3. Once we have the bucket created we need to configure it to save the terraform state files.

4. Create your `main.tf` file  and your `variable.tf` filethat will have you provider and Bucket definition. Note: the backend cannot use variables that´s why the region is write down.

`main.tf`
```
provider "aws" {
  region = "${var.region}"
}
terraform {
  backend "s3" {
    bucket = "YOUR-BUCKET-NAME"
    key    = "terraform.tfstate"
    encrypt = true
    region = "us-west-1"
  }
}

```

`variables.tf`
```
variable "region" {
  default = "us-west-1"
}

variable "environment"{
  default = "dev"
}
```

5. Initialize your backend
```
terraform init -backend-config="access_key=<YOUR ACCESS KEY>"\
-backend-config="secret_key=<YOUR SECRET KEY>"\
-backend-config="region=<REGION TO USE>"\
-backend-config="bucket=YOUR-BUCKET-NAME"\ -backend-config="key=terraform.tfstate"
```

4. Add the workspaces to use

```
terraform workspace new dev
terraform workspace new prod
terraform workspace new YOUR-WORKSPACE-NAME
```

5. Select the workspace to use

```
terraform workspace use dev
```

6. Execute you terraform plan with the env vars that you want to use

```
terraform plan --var-file="variables_dev.tfvars"
```


7. Execute you terraform apply with the env vars that you want to use

```
terraform apply --var-file="variables_dev.tfvars"
```

