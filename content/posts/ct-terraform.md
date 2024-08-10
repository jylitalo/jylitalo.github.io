---
title: "Using AWS Control Tower with terraform"
date: 2024-08-10T12:00:00+02:00
categories: ["index", "aws", "terraform"]
---
**Disclaimer**: This post is based on one customer project with pre-existing AWS Control Tower.

AWS created Control Tower as a way to allow organization to manage their AWS Organization, accounts and their setup. Control Tower has nice UI in AWS console,
but if you try to terraform anything you will quickly get confused, because terraform (with AWS provider 5.62.0) has ONLY TWO `aws_control_tower_*` resources.
Situation with awscli (version 2.17.5) isn't much better.

Here are my notes about different aspects of AWS Control Tower and how it relates to terraform resources.

## Organization Units

If you are missing some organization units from your terraform setup, you can bring them in with `terraform import aws_organizations_organizational_unit.example ou-1234567`

If you need to create new organization unit, then you have two options, but both of them require AWS Console:
- create in console and terraform import
  - create new organization unit in AWS Control Tower (**note:** in Control Tower, NOT in Organizations)
  - `terraform import aws_organizations_organizational_unit ...`
- create in terraform and enroll in console
  - create new organization with `resource "aws_organizations_organizational_unit" "example" {...}`
  - go to Control Tower in AWS Console, select new OU and Enroll OU

## Accounts

When you've setup Control Tower, you have also created 'AWS Control Tower Account Factory' into ServiceCatalog with prod-*alphanumberic* as product ID and pa-*alphanumeric* as version IDs.

You can create new accounts with `resource "aws_servicecatalog_provisioned_product" "example" {...}` in terraform, but you must define
_AccountEmail, AccountName, ManagedOrganizationalUnit, SSOUserEmail, SSOUserFirstName_ and _SSOUserLastName_ as `provisioning_parameters {...}`.
_SSOUser*_ parameters are used for creating new user into IAM IdentityCenter. This will create new account into your AWS Organization and enroll it into Control Tower.

If you need to import existing account into your terraform, things are going to be little bit more tricky. You will start with
`terraform import aws_servicecatalog_provisioned_product.example pp-dnigbtea24ste`, where pp-_alphanumberic_ can be found from AWS ServiceCatalog,
if you first select 'Provisioned products' and change Access Filter to 'Account'. It should be there with something like AccountLaunch-_account name_ or Enroll-Account-_account number_.
This is only a first step, because `terraform import` will not import necessary 'provisioning_parameter' blocks. Thankfully `aws_servicecatalog_provisioned_product` resources
output will provide _AccountId_ and _SSOUserEmail_ among other things and with that information you can for example 
- get first and last name of SSO user name: `aws identitystore list-users --identity-store-id identity_store_id --filter AttributePath=UserName,AttributeValue=sso_email`
- get organization unit by combining:
  - `aws organizations list-parents --child-id account_id`
  - `aws organizations describe-organizational-unit --organizational-unit-id ou_id`

## Blueprints

Control Tower Blueprints is one of the advanced features of Control Tower and it allows you to specify resources that new accounts should have from the beginning.
At least some of the Control Tower documentation recommends that Control Tower Blueprints should have dedicated account. Unfortunately, this brings its own layer of extra
complexity on how things work under the hood.

Just like accounts creation, Control Tower Blueprints are built on top of ServiceCatalog. However with Blueprints, the process is bit more complicated,
because Blueprints are stored as ServiceCatalog Portfolios (port-_alphanumeric_ as portfolio ID) in Blueprint account.
When you want to apply ControlTower blueprint to account _XYZ_ in AWS Console, it will
- assume AWSControlTowerExecution role in Bluetooth account
  - share selected portfolio with account _XYZ_
- assume AWSControlTowerExecution role in account _XYZ_
  - import shared portfolio into ServiceCatalog
  - select product from imported portfolio
  - launch specific version of product

At the end provisioned product should be visible in account _XYZ_ as 'Provisioned Product'.

## Conclusions

If your goal is to run your organization with GitOps or any variation of infrastructure as code, I would avoid AWS Control Tower until they have terraform resources
and awscli commands for managing accounts, organization units and blueprints, unless you really need `aws_control_tower_controls`, you have governance or compliance
requirements that say you must use it, or someone in chain of command has decided that your solution must be built on top of AWS Control Tower.

Most (if not all) of the ControlTower functionality could be built on top of AWS Organization with terraform without anything from AWS Control Tower and it would be must simpler
to maintain on a long run.
