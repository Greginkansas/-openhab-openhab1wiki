<!-- Using MarkdownTOC plugin for Sublime Text to update the table of contents (TOC) -->
<!-- MarkdownTOC depth=3 autolink=true bracket=round -->

- [Introduction](#introduction)
- [Features](#features)
- [Installation](#installation)
	- [Setting up Amazon account](#setting-up-amazon-account)
	- [Openhab configuration](#openhab-configuration)
- [Details](#details)
	- [Tables created by the addon](#tables-created-by-the-addon)
	- [Pricing](#pricing)
	- [Disclaimer](#disclaimer)

<!-- /MarkdownTOC -->


## Introduction

This service allows you to persist state updates using the Amazon DynamoDB database. Also query functionality is fully supported. Users are recommended to familiarize with AWS [Pricing](#Pricing) before using this plugin.

## Features

General:

- Writing/reading information to relational database systems.
- Database Table Name configurable
- Automatic creation of tables

## Installation

1. Set-up Amazon account as described below (see [Setting up Amazon account](#Setting up Amazon account))
2. For installation of this persistence bundle, please follow the same steps as if you would [install a binding](https://github.com/openhab/openhab/wiki/Bindings).
3. Place a persistence file called dynamodb.persist in the `${openhab.home}/configuration/persistence` folder. This has the standard format as described in [Persistence](https://github.com/openhab/openhab/wiki/Persistence).
4. Configure openhab.cfg (see [Openhab configuration](#Openhab configuration) below)


### Setting up Amazon account

1. [Sign up](https://aws.amazon.com/) for Amazon AWS
2. Select the AWS region in the [AWS console](https://console.aws.amazon.com/) using [these instructions](https://docs.aws.amazon.com/awsconsolehelpdocs/latest/gsg/getting-started.html#select-region). Note the region identifier in the url (e.g. `https://eu-west-1.console.aws.amazon.com/console/home?region=eu-west-1` means that region id is `eu-west-1`)
3. **Create user for openhab with IAM**
  1. Open Services -> IAM -> Users -> Create new Users. Enter `openhab` to _User names_, keep _Generate an access key for each user_ checked, and finally click _Create_.
  2. _Show User Security Credentials_ and record the keys displayed
4. **Configure user policy to have access for dynamodb**
  1. Open Services -> IAM -> Policies
  2. Check _AmazonDynamoDBFullAccess_ and click _Policy actions_ -> _Attach_
  3. Check the user created in step 2 and click _Attach policy_

### Openhab configuration

#### Basic configuration

Configure addon using openhab.cfg, for example

 ````ini
dynamodb:accessKey=foo
dynamodb:secretKey=bar
dynamodb:region=eu-west-1
 ````
The `accessKey` and `secretKey` correspond to credentials shown in IAM when creating the AWS user. Region needs to match the region what was used to create the user.

#### Alternative configuration using credentials file

Alternatively, instead of specifying `accessKey`, `secretKey`, one can configure `profilesConfigFile` (path to Amazon credentials file) and `profile` (name of profile to use). For example,

 ````ini
dynamodb:profilesConfigFile=/home/openhab/aws_creds
dynamodb:profile=fooprofile
dynamodb:region=eu-west-1
 ````

Example of credentials file (`/home/openhab/aws_creds`):

````ini
[fooprofile]
aws_access_key_id=testAccessKey
aws_secret_access_key=testSecretKey
````

Please note that the user that runs openhab must have approriate read rights to the credential file. For more details on the Amazon credential file format, see [Amazon documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html).

#### Advanced configuration

Other configuration parameters for the dynamodb addon:

- `readCapacityUnits`, read capacity for the created tables. Defaults to `1`.
- `writeCapacityUnits`, write capacity for the created tables. Defaults to `1`.
- `tablePrefix`, table prefix used in the name of created tables. Defaults to `openhab-`.

Refer to amazon documentation on [provisioned throughput](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ProvisionedThroughput.html) on details on read/write capacity.


## Details

### Tables created by the addon

When item is persisted via this addon, a table is created if necessary. Currently the addon will create at most two tables for different item types. The tables will be named `PREFIX-ITEMTYPE`, where PREFIX matches the `tablePrefix` configuration; while the `ITEMTYPE` is either `bigdecimal` (numeric items) or `string` (string and complex items).

Each table will have three columns: `itemname` (item name), `timeutc` (in ISO 8601 format with millisecond accuracy), and `itemstate` (either number or string representing item state).

### Pricing

Please note that there might be charges from Amazon when using this addon to query/store data to DynamoDB. See [Amazon DynamoDB pricing pages](https://aws.amazon.com/dynamodb/pricing/) for more details. Please also note possible [Free Tier](https://aws.amazon.com/free/) benefits. 

### Disclaimer

This addon is provided "AS IS", and the user takes full responsibility of any charges, damage to amazon data.