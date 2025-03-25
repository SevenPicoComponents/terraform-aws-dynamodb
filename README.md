# terraform-aws-dynamodb [![Latest Release](https://img.shields.io/github/release/sevenpicocomponents/terraform-aws-dynamodb.svg)](https://github.com/sevenpicocomponents/terraform-aws-dynamodb/releases/latest)

Terraform module to provision a DynamoDB table with autoscaling.

Autoscaler scales up/down the provisioned OPS for the DynamoDB table based on the load.

## Requirements

This module requires [AWS Provider](https://github.com/terraform-providers/terraform-provider-aws) `>= 4.22.0`

## Usage

**IMPORTANT:** We do not pin modules to versions in our examples because of the
difficulty of keeping the versions in the documentation in sync with the latest released versions.
We highly recommend that in your code you pin the version to the exact version you are
using so that your infrastructure remains stable, and update versions in a
systematic way so that they do not catch you by surprise.

Also, because of a bug in the Terraform registry ([hashicorp/terraform#21417](https://github.com/hashicorp/terraform/issues/21417)),
the registry shows many of our inputs as required when in fact they are optional.
The table below correctly indicates which inputs are required.

```hcl
module "dynamodb_table" {
  source = "sevenpicocomponents/dynamodb/aws"
  # We recommend pinning every module to a specific version
  # version     = "x.x.x"
  namespace                    = "eg"
  stage                        = "dev"
  name                         = "cluster"
  hash_key                     = "HashKey"
  range_key                    = "RangeKey"
  autoscale_write_target       = 50
  autoscale_read_target        = 50
  autoscale_min_read_capacity  = 5
  autoscale_max_read_capacity  = 20
  autoscale_min_write_capacity = 5
  autoscale_max_write_capacity = 20
  enable_autoscaler            = true
}
```

## Advanced Usage

With additional attributes, global secondary indexes and `non_key_attributes` (see [examples/complete](examples/complete)).

```hcl
module "dynamodb_table" {
  source = "sevenpicocomponents/dynamodb/aws"
  # We recommend pinning every module to a specific version
  # version     = "x.x.x"
  namespace                    = "eg"
  stage                        = "dev"
  name                         = "cluster"
  hash_key                     = "HashKey"
  range_key                    = "RangeKey"
  autoscale_write_target       = 50
  autoscale_read_target        = 50
  autoscale_min_read_capacity  = 5
  autoscale_max_read_capacity  = 20
  autoscale_min_write_capacity = 5
  autoscale_max_write_capacity = 20
  enable_autoscaler            = true

  dynamodb_attributes = [
    {
      name = "DailyAverage"
      type = "N"
    },
    {
      name = "HighWater"
      type = "N"
    },
    {
      name = "Timestamp"
      type = "S"
    }
  ]

  local_secondary_index_map = [
    {
      name               = "TimestampSortIndex"
      range_key          = "Timestamp"
      projection_type    = "INCLUDE"
      non_key_attributes = ["HashKey", "RangeKey"]
    },
    {
      name               = "HighWaterIndex"
      range_key          = "Timestamp"
      projection_type    = "INCLUDE"
      non_key_attributes = ["HashKey", "RangeKey"]
    }
  ]

  global_secondary_index_map = [
    {
      name               = "DailyAverageIndex"
      hash_key           = "DailyAverage"
      range_key          = "HighWater"
      write_capacity     = 5
      read_capacity      = 5
      projection_type    = "INCLUDE"
      non_key_attributes = ["HashKey", "RangeKey"]
    }
  ]

  replicas = ["us-east-1"]
}
```

__NOTE:__ Variables "global_secondary_index_map" and "local_secondary_index_map" have a predefined schema, but in some cases not all fields are required or needed.

For example:
 * `non_key_attributes` can't be specified for Global Secondary Indexes (GSIs) when `projection_type` is `ALL`
 * `read_capacity` and `write_capacity` are not required for GSIs

In these cases, set the fields to `null` and Terraform will treat them as if they were not provided at all, but will not complain about missing values:

```hcl
  global_secondary_index_map = [
    {
      write_capacity     = null
      read_capacity      = null
      projection_type    = "ALL"
      non_key_attributes = null
    }
  ]
```

See [Terraform types and values](https://www.terraform.io/docs/configuration/expressions.html#types-and-values) for more details.

## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 0.13.0 |
| <a name="requirement_aws"></a> [aws](#requirement\_aws) | >= 4.22 |
| <a name="requirement_null"></a> [null](#requirement\_null) | >= 2.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | >= 4.22 |
| <a name="provider_null"></a> [null](#provider\_null) | >= 2.0 |

## Modules

| Name | Source | Version |
|------|--------|---------|
| <a name="module_dynamodb_autoscaler"></a> [dynamodb\_autoscaler](#module\_dynamodb\_autoscaler) | sevenpicocomponents/dynamodb-autoscaler/aws | 0.14.0 |
| <a name="module_this"></a> [this](#module\_this) | sevenpicocomponents/label/null | 0.25.0 |

## Resources

| Name | Type |
|------|------|
| [aws_dynamodb_table.default](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/dynamodb_table) | resource |
| [null_resource.global_secondary_index_names](https://registry.terraform.io/providers/hashicorp/null/latest/docs/resources/resource) | resource |
| [null_resource.local_secondary_index_names](https://registry.terraform.io/providers/hashicorp/null/latest/docs/resources/resource) | resource |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_hash_key"></a> [hash\_key](#input\_hash\_key) | DynamoDB table Hash Key | `string` | n/a | yes |
| <a name="input_name"></a> [name](#input\_name) | Solution name, e.g. 'app' or 'jenkins' | `string` | n/a | yes |
| <a name="input_namespace"></a> [namespace](#input\_namespace) | Namespace, which could be your organization name or abbreviation, e.g. 'eg' or 'cp' | `string` | n/a | yes |
| <a name="input_stage"></a> [stage](#input\_stage) | Stage, e.g. 'prod', 'staging', 'dev', OR 'source', 'build', 'test', 'deploy', 'release' | `string` | n/a | yes |
| <a name="input_autoscale_max_read_capacity"></a> [autoscale\_max\_read\_capacity](#input\_autoscale\_max\_read\_capacity) | DynamoDB autoscaling max read capacity | `number` | `20` | no |
| <a name="input_autoscale_max_write_capacity"></a> [autoscale\_max\_write\_capacity](#input\_autoscale\_max\_write\_capacity) | DynamoDB autoscaling max write capacity | `number` | `20` | no |
| <a name="input_autoscale_min_read_capacity"></a> [autoscale\_min\_read\_capacity](#input\_autoscale\_min\_read\_capacity) | DynamoDB autoscaling min read capacity | `number` | `5` | no |
| <a name="input_autoscale_min_write_capacity"></a> [autoscale\_min\_write\_capacity](#input\_autoscale\_min\_write\_capacity) | DynamoDB autoscaling min write capacity | `number` | `5` | no |
| <a name="input_autoscale_read_target"></a> [autoscale\_read\_target](#input\_autoscale\_read\_target) | The target value (in %) for DynamoDB read autoscaling | `number` | `50` | no |
| <a name="input_autoscale_write_target"></a> [autoscale\_write\_target](#input\_autoscale\_write\_target) | The target value (in %) for DynamoDB write autoscaling | `number` | `50` | no |
| <a name="input_billing_mode"></a> [billing\_mode](#input\_billing\_mode) | DynamoDB Billing mode. Can be PROVISIONED or PAY\_PER\_REQUEST | `string` | `"PROVISIONED"` | no |
| <a name="input_dynamodb_attributes"></a> [dynamodb\_attributes](#input\_dynamodb\_attributes) | Additional DynamoDB attributes in the form of a list of mapped values | <pre>list(object({<br>    name = string<br>    type = string<br>  }))</pre> | `[]` | no |
| <a name="input_enable_autoscaler"></a> [enable\_autoscaler](#input\_enable\_autoscaler) | Flag to enable/disable DynamoDB autoscaling | `bool` | `false` | no |
| <a name="input_enable_encryption"></a> [enable\_encryption](#input\_enable\_encryption) | Enable DynamoDB server-side encryption | `bool` | `true` | no |
| <a name="input_enable_point_in_time_recovery"></a> [enable\_point\_in\_time\_recovery](#input\_enable\_point\_in\_time\_recovery) | Enable DynamoDB point in time recovery | `bool` | `true` | no |
| <a name="input_enable_streams"></a> [enable\_streams](#input\_enable\_streams) | Enable DynamoDB streams | `bool` | `false` | no |
| <a name="input_environment"></a> [environment](#input\_environment) | Environment, e.g. 'prod', 'staging', 'dev', 'UAT' | `string` | n/a | yes |
| <a name="input_global_secondary_index_map"></a> [global\_secondary\_index\_map](#input\_global\_secondary\_index\_map) | Additional global secondary indexes in the form of a list of mapped values | <pre>list(object({<br>    hash_key           = string<br>    name               = string<br>    non_key_attributes = list(string)<br>    projection_type    = string<br>    range_key          = string<br>    read_capacity      = number<br>    write_capacity     = number<br>  }))</pre> | `[]` | no |
| <a name="input_hash_key_type"></a> [hash\_key\_type](#input\_hash\_key\_type) | Hash Key type, which must be a scalar type: `S`, `N`, or `B` for (S)tring, (N)umber or (B)inary data | `string` | `"S"` | no |
| <a name="input_local_secondary_index_map"></a> [local\_secondary\_index\_map](#input\_local\_secondary\_index\_map) | Additional local secondary indexes in the form of a list of mapped values | <pre>list(object({<br>    name               = string<br>    non_key_attributes = list(string)<br>    projection_type    = string<br>    range_key          = string<br>  }))</pre> | `[]` | no |
| <a name="input_range_key"></a> [range\_key](#input\_range\_key) | DynamoDB table Range Key | `string` | `""` | no |
| <a name="input_range_key_type"></a> [range\_key\_type](#input\_range\_key\_type) | Range Key type, which must be a scalar type: `S`, `N`, or `B` for (S)tring, (N)umber or (B)inary data | `string` | `"S"` | no |
| <a name="input_replicas"></a> [replicas](#input\_replicas) | List of regions to create replica | `list(string)` | `[]` | no |
| <a name="input_server_side_encryption_kms_key_arn"></a> [server\_side\_encryption\_kms\_key\_arn](#input\_server\_side\_encryption\_kms\_key\_arn) | The ARN of the CMK that should be used for the AWS KMS encryption. This attribute should only be specified if the key is different from the default DynamoDB CMK, alias/aws/dynamodb. | `string` | `null` | no |
| <a name="input_stream_view_type"></a> [stream\_view\_type](#input\_stream\_view\_type) | When an item in the table is modified, what information is written to the stream | `string` | `""` | no |
| <a name="input_table_class"></a> [table\_class](#input\_table\_class) | DynamoDB storage class of the table. Can be STANDARD or STANDARD\_INFREQUENT\_ACCESS | `string` | `"STANDARD"` | no |
| <a name="input_tenant"></a> [tenant](#input\_tenant) | Tenant, which could be your organization name or abbreviation, e.g. 'eg' or 'cp' | `string` | n/a | yes |
| <a name="input_ttl_attribute"></a> [ttl\_attribute](#input\_ttl\_attribute) | DynamoDB table TTL attribute | `string` | `"Expires"` | no |
| <a name="input_ttl_enabled"></a> [ttl\_enabled](#input\_ttl\_enabled) | Set to false to disable DynamoDB table TTL | `bool` | `true` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_global_secondary_index_names"></a> [global\_secondary\_index\_names](#output\_global\_secondary\_index\_names) | DynamoDB secondary index names |
| <a name="output_local_secondary_index_names"></a> [local\_secondary\_index\_names](#output\_local\_secondary\_index\_names) | DynamoDB local index names |
| <a name="output_table_arn"></a> [table\_arn](#output\_table\_arn) | DynamoDB table ARN |
| <a name="output_table_id"></a> [table\_id](#output\_table\_id) | DynamoDB table ID |
| <a name="output_table_name"></a> [table\_name](#output\_table\_name) | DynamoDB table name |
| <a name="output_table_stream_arn"></a> [table\_stream\_arn](#output\_table\_stream\_arn) | DynamoDB table stream ARN |
| <a name="output_table_stream_label"></a> [table\_stream\_label](#output\_table\_stream\_label) | DynamoDB table stream label |


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
