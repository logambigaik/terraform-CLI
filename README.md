#terraform-CLI


##About Terraform CLI
Terraform, a tool created by Hashicorp in 2014, written in Go, aims to build, change and version control your infrastructure. This tool has a powerful and very intuitive Command Line Interface.

## Installation
```
Install Through curl
1
$ curl -O https://releases.hashicorp.com/terraform/0.11.10/terraform_0.11.10_linux_amd64.zip
2
$ sudo unzip terraform_0.11.10_linux_amd64.zip -d /usr/local/bin/
3
$ rm terraform_0.11.10_linux_amd64.zip


...or Install Through tfenv, a Terraform Version Manager
First of all, download the tfenv binary and put it in your PATH.

1
$ git clone https://github.com/Zordrak/tfenv.git ~/.tfenv
2
$ echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> $HOME/bashrc

```

# Then, you can install the desired version of Terraform:

```
1
$ tfenv install 0.11.10
Usage
Show Version
1
$ terraform --version
2
 Terraform v0.11.10
Init Terraform
$ terraform init

```

# It’s the first command you need to execute. Unless  terraform plan , apply  , destroy   and import  will not work. The command terraform init   will install :

```
Terraform modules
Eventually a backend
Provider(s) plugins
Init Terraform and Don’t Ask Any Input
$ terraform init -input=false

Change Backend Configuration During the Init
$ terraform init -backend-config=cfg/s3.dev.tf -reconfigure

-reconfigure is used in order to tell Terraform to not copy the existing state to the new remote state location.

Get
This command is useful when you have defined some modules. Modules are vendored, so when you edit them, you need to get again modules content.

$ terraform get -update=true

When you use modules, the first thing you’ll have to do is to do a  terraform get . This pulls modules into the .terraform directory. Once you do that, unless you do another terraform get -update=true, you’ve essentially vendored those modules.

Plan
The plan step check configuration to execute and write a plan to apply to target infrastructure provider.

$ terraform plan -out plan.out

It’s an important feature of Terraform that allows a user to see which actions Terraform will perform prior to making any changes, increasing confidence that a change will have the desired effect once applied.

When you execute terraform plan, Terraform will scan all *.tf files in your directory and create the plan.

Apply
Now you have the desired state so you can execute the plan.

$ terraform apply plan.out

Good to know: Since Terraform v0.11+, in an interactive mode (non CI/CD/autonomous pipeline), you can just execute terraform apply command which will print out which actions TF will perform.

By generating the plan and applying it in the same command, Terraform can guarantee that the execution plan won’t change, without needing to write it to disk. This reduces the risk of potentially-sensitive data being left behind, or accidentally checked into version control.

$ terraform apply

Apply and Auto Approve
$ terraform apply -auto-approve

Apply and Define New Variables Value
1
$ terraform apply -auto-approve -var tags-repository_url=${GIT_URL}


Apply Only One Module
1
$ terraform apply -target=module.s3


This -target option works with Terraform plan too.

Destroy
$ terraform destroy

Delete all the resources!

A deletion plan can be created before:

$ terraform plan –destroy

-target option allows to destroy only one resource, for example, an S3 bucket :

1
$ terraform destroy -target aws_s3_bucket.my_bucket


Debug
The terraform console  command is useful for testing interpolations before using them in configurations. Terraform console will read configured state even if it is remote.

1
$ echo "aws_iam_user.notif.arn" | terraform console
2
arn:aws:iam::123456789:user/notif


Graph
$ terraform graph | dot –Tpng > graph.png

Visual dependency graph of Terraform resources.

Validate
The validate command is used to validate/check the syntax of the Terraform files. A syntax check is done on all the Terraform files in the directory and will display an error if any of the files don’t validate. The syntax check does not cover every syntax common issues.

1
$ terraform validate


Providers
You can use a lot of providers/plugins in your Terraform definition resources, so it can be useful to have a tree of providers used by modules in your project.

1
$ terraform providers
2
.
3
├── provider.aws ~> 1.24.0
4
├── module.my_module
5
│   ├── provider.aws (inherited)
6
│   ├── provider.null
7
│   └── provider.template
8
└── module.elastic
9
    └── provider.aws (inherited)


State
Pull Remote State in A Local Copy
1
$ terraform state pull > terraform.tfstate


Push State in a Remote Backend storage
1
$ terraform state push


This command is useful if, for example, you originally use a local tf state and then you define backend storage, in S3 or Consul…

How to Tell to Terraform You Moved a Resource in A Module?
If you moved an existing resource in a module, you need to update the state:

1
$ terraform state mv aws_iam_role.role1 module.mymodule


How to Import Existing Resource in Terraform?
If you have an existing resource in your infrastructure provider, you can import it in your Terraform state:

1
$ terraform import aws_iam_policy.elastic_post
2
arn:aws:iam::123456789:policy/elastic_post


Workspaces
To manage multiple distinct sets of infrastructure resources/environments.

Instead of creating a directory for each environment to manage, we need to just create needed workspace and use them:

Create Workspace
This command creates a new workspace and then select it

$ terraform workspace new dev

Select a Workspace
$ terraform workspace select dev

List Workspaces


1
$ terraform workspace list
2
  default
3
* dev
4
  preprod


Show Current Workspace


1
$ terraform workspace show
2
dev


Tools
jq
jq is a lightweight command-line JSON processor. Combined with Terraform output it can be powerful.

Installation
For Linux:

$ sudo apt-get install jq

or

$ yum install jq

For OS X:

$ brew install jq

Usage
For example, we defined outputs in a module and when we execute terraform apply outputs are displayed:



1
$ terraform apply
2
...
3
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
4
​
5
Outputs:
6
​
7
elastic_endpoint = vpc-toto-12fgfd4d5f4ds5fngetwe4.eu-central-1.es.amazonaws.com


We can extract the value that we want in order to use it in a script for example. With jq it’s easy:

1
$ terraform output -json
2
{
3
    "elastic_endpoint": {
4
        "sensitive": false,
5
        "type": "string",
6
        "value": "vpc-toto-12fgfd4d5f4ds5fngetwe4.eu-central-1.es.amazonaws.com"
7
    }
8
}
9
​
10
$ terraform output -json | jq '.elastic_endpoint.value'
11
"vpc-toto-12fgfd4d5f4ds5fngetwe4.eu-central-1.es.amazonaws.com"


Terraforming
If you have an existing AWS accountl for examples with existing components like S3 buckets, SNS, VPC … You can use Terraforming tool, a tool written in Ruby, which extracts existing AWS resources and converts it to Terraform files!

Installation
$ sudo apt install ruby or $ sudo yum install ruby

and

1
$ gem install terraforming


Usage
Pre-requisites:
Like for Terraform, you need to set AWS credentials:

1
$ export AWS_ACCESS_KEY_ID="an_aws_access_key"
2
$ export AWS_SECRET_ACCESS_KEY="a_aws_secret_key"
3
$ export AWS_DEFAULT_REGION="eu-central-1"


You can also specify credential profile in ~/.aws/credentials_s and with _–profile option.

1
$ cat ~/.aws/credentials
2
[aurelie]
3
aws_access_key_id = xxx
4
aws_secret_access_key = xxx
5
aws_default_region = eu-central-1


1
$ terraforming s3 --profile aurelie


Usage
1
$ terraforming --help
2
Commands:
3
terraforming alb # ALB
4
...
5
terraforming vgw # VPN Gateway
6
terraforming vpc # VPC


Example:

$ terraforming s3 > aws_s3.tf


```
