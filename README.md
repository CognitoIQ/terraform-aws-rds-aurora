# WARNING: THIS IS A PUBLIC REPO THAT HAS BEEN FORKED. DO NOT INCLUDE SENSITIVE INFORMATION IN ANY COMMITS.

# AWS RDS Aurora Terraform module

Terraform module which creates AWS RDS Aurora resources.

[![SWUbanner](https://raw.githubusercontent.com/vshymanskyy/StandWithUkraine/main/banner2-direct.svg)](https://github.com/vshymanskyy/StandWithUkraine/blob/main/docs/README.md)

## Available Features

- Autoscaling of read-replicas
- Global cluster
- Enhanced monitoring
- Serverless cluster (v1 and v2)
- Import from S3
- Fine grained control of individual cluster instances
- Custom endpoints

## Usage

```hcl
module "cluster" {
  source  = "terraform-aws-modules/rds-aurora/aws"

  name           = "test-aurora-db-postgres96"
  engine         = "aurora-postgresql"
  engine_version = "11.12"
  instance_class = "db.r6g.large"
  instances = {
    one = {}
    2 = {
      instance_class = "db.r6g.2xlarge"
    }
  }

  vpc_id  = "vpc-12345678"
  subnets = ["subnet-12345678", "subnet-87654321"]

  allowed_security_groups = ["sg-12345678"]
  allowed_cidr_blocks     = ["10.20.0.0/20"]

  storage_encrypted   = true
  apply_immediately   = true
  monitoring_interval = 10

  db_parameter_group_name         = "default"
  db_cluster_parameter_group_name = "default"

  enabled_cloudwatch_logs_exports = ["postgresql"]

  tags = {
    Environment = "dev"
    Terraform   = "true"
  }
}
```

### Cluster Instance Configuration

There are a couple different configuration methods that can be used to create instances within the cluster:

ℹ️ Only the pertinent attributes are shown for brevity

1. Create homogenous cluster of any number of instances

- Resources created:
  - Writer: 1
  - Reader(s): 2

```hcl
  instance_class = "db.r6g.large"
  instances = {
    one   = {}
    two   = {}
    three = {}
  }
```

2. Create homogenous cluster of instances w/ autoscaling enabled. This is redundant and we'll show why in the next example.

- Resources created:
  - Writer: 1
  - Reader(s):
    - At least 4 readers (2 created directly, 2 created by appautoscaling)
    - At most 7 reader instances (2 created directly, 5 created by appautoscaling)

ℹ️ Autoscaling uses the instance class specified by `instance_class`.

```hcl
  instance_class = "db.r6g.large"
  instances = {
    one   = {}
    two   = {}
    three = {}
  }

  autoscaling_enabled      = true
  autoscaling_min_capacity = 2
  autoscaling_max_capacity = 5
```

3. Create homogeneous cluster scaled via autoscaling. At least one instance (writer) is required

- Resources created:
  - Writer: 1
  - Reader(s):
    - At least 1 reader
    - At most 5 readers

```hcl
  instance_class = "db.r6g.large"
  instances = {
    one = {}
  }

  autoscaling_enabled      = true
  autoscaling_min_capacity = 1
  autoscaling_max_capacity = 5
```

4. Create heterogenous cluster to support mixed-use workloads

   It is common in this configuration to independently control the instance `promotion_tier` paired with `endpoints` to create custom endpoints directed at select instances or instance groups.

- Resources created:
  - Writer: 1
  - Readers: 2

```hcl
  instance_class = "db.r5.large"
  instances = {
    one = {
      instance_class      = "db.r5.2xlarge"
      publicly_accessible = true
    }
    two = {
      identifier     = "static-member-1"
      instance_class = "db.r5.2xlarge"
    }
    three = {
      identifier     = "excluded-member-1"
      instance_class = "db.r5.large"
      promotion_tier = 15
    }
  }
```

5. Create heterogenous cluster to support mixed-use workloads w/ autoscaling enabled

- Resources created:
  - Writer: 1
  - Reader(s):
    - At least 3 readers (2 created directly, 1 created through appautoscaling)
    - At most 7 readers (2 created directly, 5 created through appautoscaling)

ℹ️ Autoscaling uses the instance class specified by `instance_class`.

```hcl
  instance_class = "db.r5.large"
  instances = {
    one = {
      instance_class      = "db.r5.2xlarge"
      publicly_accessible = true
    }
    two = {
      identifier     = "static-member-1"
      instance_class = "db.r5.2xlarge"
    }
    three = {
      identifier     = "excluded-member-1"
      instance_class = "db.r5.large"
      promotion_tier = 15
    }
  }

  autoscaling_enabled      = true
  autoscaling_min_capacity = 1
  autoscaling_max_capacity = 5
```

## Conditional Creation

The following values are provided to toggle on/off creation of the associated resources as desired:

```hcl
# This RDS cluster will not be created
module "cluster" {
  source  = "terraform-aws-modules/rds-aurora/aws"

  # Disable creation of cluster and all resources
  create_cluster = false

  # Disable creation of subnet group - provide a subnet group
  create_db_subnet_group = false

  # Disable creation of security group - provide a security group
  create_security_group = false

  # Disable creation of monitoring IAM role - provide a role ARN
  create_monitoring_role = false

  # Disable creation of random password - AWS API provides the password
  create_random_password = false

  # ... omitted
}
```

## Examples

- [Autoscaling](https://github.com/terraform-aws-modules/terraform-aws-rds-aurora/tree/master/examples/autoscaling): A PostgreSQL cluster with enhanced monitoring and autoscaling enabled
- [Global Cluster](https://github.com/terraform-aws-modules/terraform-aws-rds-aurora/tree/master/examples/global_cluster): A PostgreSQL global cluster with clusters provisioned in two different region
- [MySQL](https://github.com/terraform-aws-modules/terraform-aws-rds-aurora/tree/master/examples/mysql): A simple MySQL cluster
- [PostgreSQL](https://github.com/terraform-aws-modules/terraform-aws-rds-aurora/tree/master/examples/postgresql): A simple PostgreSQL cluster
- [S3 Import](https://github.com/terraform-aws-modules/terraform-aws-rds-aurora/tree/master/examples/s3_import): A MySQL cluster created from a Percona Xtrabackup stored in S3
- [Serverless](https://github.com/terraform-aws-modules/terraform-aws-rds-aurora/tree/master/examples/serverless): Serverless V1 and V2 (PostgreSQL and MySQL)

## Documentation

Terraform documentation is generated automatically using [pre-commit hooks](http://www.pre-commit.com/). Follow installation instructions [here](https://pre-commit.com/#install).

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->

## Requirements

| Name                                                                     | Version |
| ------------------------------------------------------------------------ | ------- |
| <a name="requirement_terraform"></a> [terraform](#requirement_terraform) | >= 0.13 |
| <a name="requirement_aws"></a> [aws](#requirement_aws)                   | >= 4.12 |
| <a name="requirement_random"></a> [random](#requirement_random)          | >= 2.2  |

## Providers

| Name                                                      | Version |
| --------------------------------------------------------- | ------- |
| <a name="provider_aws"></a> [aws](#provider_aws)          | >= 4.12 |
| <a name="provider_random"></a> [random](#provider_random) | >= 2.2  |

## Modules

No modules.

## Resources

| Name                                                                                                                                                             | Type        |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| [aws_appautoscaling_policy.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/appautoscaling_policy)                              | resource    |
| [aws_appautoscaling_target.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/appautoscaling_target)                              | resource    |
| [aws_db_parameter_group.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/db_parameter_group)                                    | resource    |
| [aws_db_subnet_group.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/db_subnet_group)                                          | resource    |
| [aws_iam_role.rds_enhanced_monitoring](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role)                                     | resource    |
| [aws_iam_role_policy_attachment.rds_enhanced_monitoring](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role_policy_attachment) | resource    |
| [aws_rds_cluster.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/rds_cluster)                                                  | resource    |
| [aws_rds_cluster_endpoint.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/rds_cluster_endpoint)                                | resource    |
| [aws_rds_cluster_instance.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/rds_cluster_instance)                                | resource    |
| [aws_rds_cluster_parameter_group.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/rds_cluster_parameter_group)                  | resource    |
| [aws_rds_cluster_role_association.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/rds_cluster_role_association)                | resource    |
| [aws_security_group.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group)                                            | resource    |
| [aws_security_group_rule.cidr_ingress](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule)                          | resource    |
| [aws_security_group_rule.default_ingress](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule)                       | resource    |
| [aws_security_group_rule.egress](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule)                                | resource    |
| [random_id.snapshot_identifier](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/id)                                               | resource    |
| [random_password.master_password](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password)                                       | resource    |
| [aws_iam_policy_document.monitoring_rds_assume_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/iam_policy_document)         | data source |
| [aws_partition.current](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/partition)                                                | data source |

## Inputs

| Name                                                                                                                                                               | Description                                                                                                                                                                                   | Type                | Default                            | Required |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------- | ---------------------------------- | :------: |
| <a name="input_allow_major_version_upgrade"></a> [allow_major_version_upgrade](#input_allow_major_version_upgrade)                                                 | Enable to allow major engine version upgrades when changing engine versions. Defaults to `false`                                                                                              | `bool`              | `false`                            |    no    |
| <a name="input_allowed_cidr_blocks"></a> [allowed_cidr_blocks](#input_allowed_cidr_blocks)                                                                         | A list of CIDR blocks which are allowed to access the database                                                                                                                                | `list(string)`      | `[]`                               |    no    |
| <a name="input_allowed_security_groups"></a> [allowed_security_groups](#input_allowed_security_groups)                                                             | A list of Security Group ID's to allow access to                                                                                                                                              | `list(string)`      | `[]`                               |    no    |
| <a name="input_apply_immediately"></a> [apply_immediately](#input_apply_immediately)                                                                               | Specifies whether any cluster modifications are applied immediately, or during the next maintenance window. Default is `false`                                                                | `bool`              | `null`                             |    no    |
| <a name="input_auto_minor_version_upgrade"></a> [auto_minor_version_upgrade](#input_auto_minor_version_upgrade)                                                    | Indicates that minor engine upgrades will be applied automatically to the DB instance during the maintenance window. Default `true`                                                           | `bool`              | `null`                             |    no    |
| <a name="input_autoscaling_enabled"></a> [autoscaling_enabled](#input_autoscaling_enabled)                                                                         | Determines whether autoscaling of the cluster read replicas is enabled                                                                                                                        | `bool`              | `false`                            |    no    |
| <a name="input_autoscaling_max_capacity"></a> [autoscaling_max_capacity](#input_autoscaling_max_capacity)                                                          | Maximum number of read replicas permitted when autoscaling is enabled                                                                                                                         | `number`            | `2`                                |    no    |
| <a name="input_autoscaling_min_capacity"></a> [autoscaling_min_capacity](#input_autoscaling_min_capacity)                                                          | Minimum number of read replicas permitted when autoscaling is enabled                                                                                                                         | `number`            | `0`                                |    no    |
| <a name="input_autoscaling_policy_name"></a> [autoscaling_policy_name](#input_autoscaling_policy_name)                                                             | Autoscaling policy name                                                                                                                                                                       | `string`            | `"target-metric"`                  |    no    |
| <a name="input_autoscaling_scale_in_cooldown"></a> [autoscaling_scale_in_cooldown](#input_autoscaling_scale_in_cooldown)                                           | Cooldown in seconds before allowing further scaling operations after a scale in                                                                                                               | `number`            | `300`                              |    no    |
| <a name="input_autoscaling_scale_out_cooldown"></a> [autoscaling_scale_out_cooldown](#input_autoscaling_scale_out_cooldown)                                        | Cooldown in seconds before allowing further scaling operations after a scale out                                                                                                              | `number`            | `300`                              |    no    |
| <a name="input_autoscaling_target_connections"></a> [autoscaling_target_connections](#input_autoscaling_target_connections)                                        | Average number of connections threshold which will initiate autoscaling. Default value is 70% of db.r4/r5/r6g.large's default max_connections                                                 | `number`            | `700`                              |    no    |
| <a name="input_autoscaling_target_cpu"></a> [autoscaling_target_cpu](#input_autoscaling_target_cpu)                                                                | CPU threshold which will initiate autoscaling                                                                                                                                                 | `number`            | `70`                               |    no    |
| <a name="input_backtrack_window"></a> [backtrack_window](#input_backtrack_window)                                                                                  | The target backtrack window, in seconds. Only available for `aurora` engine currently. To disable backtracking, set this value to 0. Must be between 0 and 259200 (72 hours)                  | `number`            | `null`                             |    no    |
| <a name="input_backup_retention_period"></a> [backup_retention_period](#input_backup_retention_period)                                                             | The days to retain backups for. Default `7`                                                                                                                                                   | `number`            | `7`                                |    no    |
| <a name="input_ca_cert_identifier"></a> [ca_cert_identifier](#input_ca_cert_identifier)                                                                            | The identifier of the CA certificate for the DB instance                                                                                                                                      | `string`            | `null`                             |    no    |
| <a name="input_cluster_tags"></a> [cluster_tags](#input_cluster_tags)                                                                                              | A map of tags to add to only the cluster. Used for AWS Instance Scheduler tagging                                                                                                             | `map(string)`       | `{}`                               |    no    |
| <a name="input_cluster_timeouts"></a> [cluster_timeouts](#input_cluster_timeouts)                                                                                  | Create, update, and delete timeout configurations for the cluster                                                                                                                             | `map(string)`       | `{}`                               |    no    |
| <a name="input_copy_tags_to_snapshot"></a> [copy_tags_to_snapshot](#input_copy_tags_to_snapshot)                                                                   | Copy all Cluster `tags` to snapshots                                                                                                                                                          | `bool`              | `null`                             |    no    |
| <a name="input_create_cluster"></a> [create_cluster](#input_create_cluster)                                                                                        | Whether cluster should be created (affects nearly all resources)                                                                                                                              | `bool`              | `true`                             |    no    |
| <a name="input_create_db_cluster_parameter_group"></a> [create_db_cluster_parameter_group](#input_create_db_cluster_parameter_group)                               | Determines whether a cluster parameter should be created or use existing                                                                                                                      | `bool`              | `false`                            |    no    |
| <a name="input_create_db_parameter_group"></a> [create_db_parameter_group](#input_create_db_parameter_group)                                                       | Determines whether a DB parameter should be created or use existing                                                                                                                           | `bool`              | `false`                            |    no    |
| <a name="input_create_db_subnet_group"></a> [create_db_subnet_group](#input_create_db_subnet_group)                                                                | Determines whether to create the database subnet group or use existing                                                                                                                        | `bool`              | `true`                             |    no    |
| <a name="input_create_monitoring_role"></a> [create_monitoring_role](#input_create_monitoring_role)                                                                | Determines whether to create the IAM role for RDS enhanced monitoring                                                                                                                         | `bool`              | `true`                             |    no    |
| <a name="input_create_random_password"></a> [create_random_password](#input_create_random_password)                                                                | Determines whether to create random password for RDS primary cluster                                                                                                                          | `bool`              | `true`                             |    no    |
| <a name="input_create_security_group"></a> [create_security_group](#input_create_security_group)                                                                   | Determines whether to create security group for RDS cluster                                                                                                                                   | `bool`              | `true`                             |    no    |
| <a name="input_database_name"></a> [database_name](#input_database_name)                                                                                           | Name for an automatically created database on cluster creation                                                                                                                                | `string`            | `null`                             |    no    |
| <a name="input_db_cluster_db_instance_parameter_group_name"></a> [db_cluster_db_instance_parameter_group_name](#input_db_cluster_db_instance_parameter_group_name) | Instance parameter group to associate with all instances of the DB cluster. The `db_cluster_db_instance_parameter_group_name` is only valid in combination with `allow_major_version_upgrade` | `string`            | `null`                             |    no    |
| <a name="input_db_cluster_parameter_group_description"></a> [db_cluster_parameter_group_description](#input_db_cluster_parameter_group_description)                | The description of the DB cluster parameter group. Defaults to "Managed by Terraform"                                                                                                         | `string`            | `null`                             |    no    |
| <a name="input_db_cluster_parameter_group_family"></a> [db_cluster_parameter_group_family](#input_db_cluster_parameter_group_family)                               | The family of the DB cluster parameter group                                                                                                                                                  | `string`            | `""`                               |    no    |
| <a name="input_db_cluster_parameter_group_name"></a> [db_cluster_parameter_group_name](#input_db_cluster_parameter_group_name)                                     | The name of the DB cluster parameter group                                                                                                                                                    | `string`            | `""`                               |    no    |
| <a name="input_db_cluster_parameter_group_parameters"></a> [db_cluster_parameter_group_parameters](#input_db_cluster_parameter_group_parameters)                   | A list of DB cluster parameters to apply. Note that parameters may differ from a family to an other                                                                                           | `list(map(string))` | `[]`                               |    no    |
| <a name="input_db_cluster_parameter_group_use_name_prefix"></a> [db_cluster_parameter_group_use_name_prefix](#input_db_cluster_parameter_group_use_name_prefix)    | Determines whether the DB cluster parameter group name is used as a prefix                                                                                                                    | `bool`              | `true`                             |    no    |
| <a name="input_db_parameter_group_description"></a> [db_parameter_group_description](#input_db_parameter_group_description)                                        | The description of the DB parameter group. Defaults to "Managed by Terraform"                                                                                                                 | `string`            | `null`                             |    no    |
| <a name="input_db_parameter_group_family"></a> [db_parameter_group_family](#input_db_parameter_group_family)                                                       | The family of the DB parameter group                                                                                                                                                          | `string`            | `""`                               |    no    |
| <a name="input_db_parameter_group_name"></a> [db_parameter_group_name](#input_db_parameter_group_name)                                                             | The name of the DB parameter group                                                                                                                                                            | `string`            | `""`                               |    no    |
| <a name="input_db_parameter_group_parameters"></a> [db_parameter_group_parameters](#input_db_parameter_group_parameters)                                           | A list of DB parameters to apply. Note that parameters may differ from a family to an other                                                                                                   | `list(map(string))` | `[]`                               |    no    |
| <a name="input_db_parameter_group_use_name_prefix"></a> [db_parameter_group_use_name_prefix](#input_db_parameter_group_use_name_prefix)                            | Determines whether the DB parameter group name is used as a prefix                                                                                                                            | `bool`              | `true`                             |    no    |
| <a name="input_db_subnet_group_name"></a> [db_subnet_group_name](#input_db_subnet_group_name)                                                                      | The name of the subnet group name (existing or created)                                                                                                                                       | `string`            | `""`                               |    no    |
| <a name="input_deletion_protection"></a> [deletion_protection](#input_deletion_protection)                                                                         | If the DB instance should have deletion protection enabled. The database can't be deleted when this value is set to `true`. The default is `false`                                            | `bool`              | `null`                             |    no    |
| <a name="input_enable_global_write_forwarding"></a> [enable_global_write_forwarding](#input_enable_global_write_forwarding)                                        | Whether cluster should forward writes to an associated global cluster. Applied to secondary clusters to enable them to forward writes to an `aws_rds_global_cluster`'s primary cluster        | `bool`              | `null`                             |    no    |
| <a name="input_enable_http_endpoint"></a> [enable_http_endpoint](#input_enable_http_endpoint)                                                                      | Enable HTTP endpoint (data API). Only valid when engine_mode is set to `serverless`                                                                                                           | `bool`              | `null`                             |    no    |
| <a name="input_enabled_cloudwatch_logs_exports"></a> [enabled_cloudwatch_logs_exports](#input_enabled_cloudwatch_logs_exports)                                     | Set of log types to export to cloudwatch. If omitted, no logs will be exported. The following log types are supported: `audit`, `error`, `general`, `slowquery`, `postgresql`                 | `list(string)`      | `[]`                               |    no    |
| <a name="input_endpoints"></a> [endpoints](#input_endpoints)                                                                                                       | Map of additional cluster endpoints and their attributes to be created                                                                                                                        | `any`               | `{}`                               |    no    |
| <a name="input_engine"></a> [engine](#input_engine)                                                                                                                | The name of the database engine to be used for this DB cluster. Defaults to `aurora`. Valid Values: `aurora`, `aurora-mysql`, `aurora-postgresql`                                             | `string`            | `null`                             |    no    |
| <a name="input_engine_mode"></a> [engine_mode](#input_engine_mode)                                                                                                 | The database engine mode. Valid values: `global`, `multimaster`, `parallelquery`, `provisioned`, `serverless`. Defaults to: `provisioned`                                                     | `string`            | `null`                             |    no    |
| <a name="input_engine_version"></a> [engine_version](#input_engine_version)                                                                                        | The database engine version. Updating this argument results in an outage                                                                                                                      | `string`            | `null`                             |    no    |
| <a name="input_final_snapshot_identifier_prefix"></a> [final_snapshot_identifier_prefix](#input_final_snapshot_identifier_prefix)                                  | The prefix name to use when creating a final snapshot on cluster destroy; a 8 random digits are appended to name to ensure it's unique                                                        | `string`            | `"final"`                          |    no    |
| <a name="input_global_cluster_identifier"></a> [global_cluster_identifier](#input_global_cluster_identifier)                                                       | The global cluster identifier specified on `aws_rds_global_cluster`                                                                                                                           | `string`            | `null`                             |    no    |
| <a name="input_iam_database_authentication_enabled"></a> [iam_database_authentication_enabled](#input_iam_database_authentication_enabled)                         | Specifies whether or mappings of AWS Identity and Access Management (IAM) accounts to database accounts is enabled                                                                            | `bool`              | `null`                             |    no    |
| <a name="input_iam_role_description"></a> [iam_role_description](#input_iam_role_description)                                                                      | Description of the monitoring role                                                                                                                                                            | `string`            | `null`                             |    no    |
| <a name="input_iam_role_force_detach_policies"></a> [iam_role_force_detach_policies](#input_iam_role_force_detach_policies)                                        | Whether to force detaching any policies the monitoring role has before destroying it                                                                                                          | `bool`              | `null`                             |    no    |
| <a name="input_iam_role_managed_policy_arns"></a> [iam_role_managed_policy_arns](#input_iam_role_managed_policy_arns)                                              | Set of exclusive IAM managed policy ARNs to attach to the monitoring role                                                                                                                     | `list(string)`      | `null`                             |    no    |
| <a name="input_iam_role_max_session_duration"></a> [iam_role_max_session_duration](#input_iam_role_max_session_duration)                                           | Maximum session duration (in seconds) that you want to set for the monitoring role                                                                                                            | `number`            | `null`                             |    no    |
| <a name="input_iam_role_name"></a> [iam_role_name](#input_iam_role_name)                                                                                           | Friendly name of the monitoring role                                                                                                                                                          | `string`            | `null`                             |    no    |
| <a name="input_iam_role_path"></a> [iam_role_path](#input_iam_role_path)                                                                                           | Path for the monitoring role                                                                                                                                                                  | `string`            | `null`                             |    no    |
| <a name="input_iam_role_permissions_boundary"></a> [iam_role_permissions_boundary](#input_iam_role_permissions_boundary)                                           | The ARN of the policy that is used to set the permissions boundary for the monitoring role                                                                                                    | `string`            | `null`                             |    no    |
| <a name="input_iam_role_use_name_prefix"></a> [iam_role_use_name_prefix](#input_iam_role_use_name_prefix)                                                          | Determines whether to use `iam_role_name` as is or create a unique name beginning with the `iam_role_name` as the prefix                                                                      | `bool`              | `false`                            |    no    |
| <a name="input_iam_roles"></a> [iam_roles](#input_iam_roles)                                                                                                       | Map of IAM roles and supported feature names to associate with the cluster                                                                                                                    | `map(map(string))`  | `{}`                               |    no    |
| <a name="input_instance_class"></a> [instance_class](#input_instance_class)                                                                                        | Instance type to use at master instance. Note: if `autoscaling_enabled` is `true`, this will be the same instance class used on instances created by autoscaling                              | `string`            | `""`                               |    no    |
| <a name="input_instance_timeouts"></a> [instance_timeouts](#input_instance_timeouts)                                                                               | Create, update, and delete timeout configurations for the cluster instance(s)                                                                                                                 | `map(string)`       | `{}`                               |    no    |
| <a name="input_instances"></a> [instances](#input_instances)                                                                                                       | Map of cluster instances and any specific/overriding attributes to be created                                                                                                                 | `any`               | `{}`                               |    no    |
| <a name="input_instances_use_identifier_prefix"></a> [instances_use_identifier_prefix](#input_instances_use_identifier_prefix)                                     | Determines whether cluster instance identifiers are used as prefixes                                                                                                                          | `bool`              | `false`                            |    no    |
| <a name="input_is_primary_cluster"></a> [is_primary_cluster](#input_is_primary_cluster)                                                                            | Determines whether cluster is primary cluster with writer instance (set to `false` for global cluster and replica clusters)                                                                   | `bool`              | `true`                             |    no    |
| <a name="input_kms_key_id"></a> [kms_key_id](#input_kms_key_id)                                                                                                    | The ARN for the KMS encryption key. When specifying `kms_key_id`, `storage_encrypted` needs to be set to `true`                                                                               | `string`            | `null`                             |    no    |
| <a name="input_master_password"></a> [master_password](#input_master_password)                                                                                     | Password for the master DB user. Note - when specifying a value here, 'create_random_password' should be set to `false`                                                                       | `string`            | `null`                             |    no    |
| <a name="input_master_username"></a> [master_username](#input_master_username)                                                                                     | Username for the master DB user                                                                                                                                                               | `string`            | `"root"`                           |    no    |
| <a name="input_monitoring_interval"></a> [monitoring_interval](#input_monitoring_interval)                                                                         | The interval, in seconds, between points when Enhanced Monitoring metrics are collected for instances. Set to `0` to disble. Default is `0`                                                   | `number`            | `0`                                |    no    |
| <a name="input_monitoring_role_arn"></a> [monitoring_role_arn](#input_monitoring_role_arn)                                                                         | IAM role used by RDS to send enhanced monitoring metrics to CloudWatch                                                                                                                        | `string`            | `""`                               |    no    |
| <a name="input_name"></a> [name](#input_name)                                                                                                                      | Name used across resources created                                                                                                                                                            | `string`            | `""`                               |    no    |
| <a name="input_performance_insights_enabled"></a> [performance_insights_enabled](#input_performance_insights_enabled)                                              | Specifies whether Performance Insights is enabled or not                                                                                                                                      | `bool`              | `null`                             |    no    |
| <a name="input_performance_insights_kms_key_id"></a> [performance_insights_kms_key_id](#input_performance_insights_kms_key_id)                                     | The ARN for the KMS key to encrypt Performance Insights data                                                                                                                                  | `string`            | `null`                             |    no    |
| <a name="input_performance_insights_retention_period"></a> [performance_insights_retention_period](#input_performance_insights_retention_period)                   | Amount of time in days to retain Performance Insights data. Either 7 (7 days) or 731 (2 years)                                                                                                | `number`            | `null`                             |    no    |
| <a name="input_port"></a> [port](#input_port)                                                                                                                      | The port on which the DB accepts connections                                                                                                                                                  | `string`            | `null`                             |    no    |
| <a name="input_predefined_metric_type"></a> [predefined_metric_type](#input_predefined_metric_type)                                                                | The metric type to scale on. Valid values are `RDSReaderAverageCPUUtilization` and `RDSReaderAverageDatabaseConnections`                                                                      | `string`            | `"RDSReaderAverageCPUUtilization"` |    no    |
| <a name="input_preferred_backup_window"></a> [preferred_backup_window](#input_preferred_backup_window)                                                             | The daily time range during which automated backups are created if automated backups are enabled using the `backup_retention_period` parameter. Time in UTC                                   | `string`            | `"02:00-03:00"`                    |    no    |
| <a name="input_preferred_maintenance_window"></a> [preferred_maintenance_window](#input_preferred_maintenance_window)                                              | The weekly time range during which system maintenance can occur, in (UTC)                                                                                                                     | `string`            | `"sun:05:00-sun:06:00"`            |    no    |
| <a name="input_publicly_accessible"></a> [publicly_accessible](#input_publicly_accessible)                                                                         | Determines whether instances are publicly accessible. Default false                                                                                                                           | `bool`              | `null`                             |    no    |
| <a name="input_putin_khuylo"></a> [putin_khuylo](#input_putin_khuylo)                                                                                              | Do you agree that Putin doesn't respect Ukrainian sovereignty and territorial integrity? More info: https://en.wikipedia.org/wiki/Putin_khuylo!                                               | `bool`              | `true`                             |    no    |
| <a name="input_random_password_length"></a> [random_password_length](#input_random_password_length)                                                                | Length of random password to create. Defaults to `10`                                                                                                                                         | `number`            | `10`                               |    no    |
| <a name="input_replication_source_identifier"></a> [replication_source_identifier](#input_replication_source_identifier)                                           | ARN of a source DB cluster or DB instance if this DB cluster is to be created as a Read Replica                                                                                               | `string`            | `null`                             |    no    |
| <a name="input_restore_to_point_in_time"></a> [restore_to_point_in_time](#input_restore_to_point_in_time)                                                          | Map of nested attributes for cloning Aurora cluster                                                                                                                                           | `map(string)`       | `{}`                               |    no    |
| <a name="input_s3_import"></a> [s3_import](#input_s3_import)                                                                                                       | Configuration map used to restore from a Percona Xtrabackup in S3 (only MySQL is supported)                                                                                                   | `map(string)`       | `null`                             |    no    |
| <a name="input_scaling_configuration"></a> [scaling_configuration](#input_scaling_configuration)                                                                   | Map of nested attributes with scaling properties. Only valid when `engine_mode` is set to `serverless`                                                                                        | `map(string)`       | `{}`                               |    no    |
| <a name="input_security_group_description"></a> [security_group_description](#input_security_group_description)                                                    | The description of the security group. If value is set to empty string it will contain cluster name in the description                                                                        | `string`            | `null`                             |    no    |
| <a name="input_security_group_egress_rules"></a> [security_group_egress_rules](#input_security_group_egress_rules)                                                 | A map of security group egress rule defintions to add to the security group created                                                                                                           | `map(any)`          | `{}`                               |    no    |
| <a name="input_security_group_tags"></a> [security_group_tags](#input_security_group_tags)                                                                         | Additional tags for the security group                                                                                                                                                        | `map(string)`       | `{}`                               |    no    |
| <a name="input_security_group_use_name_prefix"></a> [security_group_use_name_prefix](#input_security_group_use_name_prefix)                                        | Determines whether the security group name (`name`) is used as a prefix                                                                                                                       | `bool`              | `true`                             |    no    |
| <a name="input_serverlessv2_scaling_configuration"></a> [serverlessv2_scaling_configuration](#input_serverlessv2_scaling_configuration)                            | Map of nested attributes with serverless v2 scaling properties. Only valid when `engine_mode` is set to `provisioned`                                                                         | `map(string)`       | `{}`                               |    no    |
| <a name="input_skip_final_snapshot"></a> [skip_final_snapshot](#input_skip_final_snapshot)                                                                         | Determines whether a final snapshot is created before the cluster is deleted. If true is specified, no snapshot is created                                                                    | `bool`              | `null`                             |    no    |
| <a name="input_snapshot_identifier"></a> [snapshot_identifier](#input_snapshot_identifier)                                                                         | Specifies whether or not to create this cluster from a snapshot. You can use either the name or ARN when specifying a DB cluster snapshot, or the ARN when specifying a DB snapshot           | `string`            | `null`                             |    no    |
| <a name="input_source_region"></a> [source_region](#input_source_region)                                                                                           | The source region for an encrypted replica DB cluster                                                                                                                                         | `string`            | `null`                             |    no    |
| <a name="input_storage_encrypted"></a> [storage_encrypted](#input_storage_encrypted)                                                                               | Specifies whether the DB cluster is encrypted. The default is `true`                                                                                                                          | `bool`              | `true`                             |    no    |
| <a name="input_subnets"></a> [subnets](#input_subnets)                                                                                                             | List of subnet IDs used by database subnet group created                                                                                                                                      | `list(string)`      | `[]`                               |    no    |
| <a name="input_tags"></a> [tags](#input_tags)                                                                                                                      | A map of tags to add to all resources                                                                                                                                                         | `map(string)`       | `{}`                               |    no    |
| <a name="input_vpc_id"></a> [vpc_id](#input_vpc_id)                                                                                                                | ID of the VPC where to create security group                                                                                                                                                  | `string`            | `""`                               |    no    |
| <a name="input_vpc_security_group_ids"></a> [vpc_security_group_ids](#input_vpc_security_group_ids)                                                                | List of VPC security groups to associate to the cluster in addition to the SG we create in this module                                                                                        | `list(string)`      | `[]`                               |    no    |

## Outputs

| Name                                                                                                                                                  | Description                                                                       |
| ----------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| <a name="output_additional_cluster_endpoints"></a> [additional_cluster_endpoints](#output_additional_cluster_endpoints)                               | A map of additional cluster endpoints and their attributes                        |
| <a name="output_cluster_arn"></a> [cluster_arn](#output_cluster_arn)                                                                                  | Amazon Resource Name (ARN) of cluster                                             |
| <a name="output_cluster_database_name"></a> [cluster_database_name](#output_cluster_database_name)                                                    | Name for an automatically created database on cluster creation                    |
| <a name="output_cluster_endpoint"></a> [cluster_endpoint](#output_cluster_endpoint)                                                                   | Writer endpoint for the cluster                                                   |
| <a name="output_cluster_engine_version_actual"></a> [cluster_engine_version_actual](#output_cluster_engine_version_actual)                            | The running version of the cluster database                                       |
| <a name="output_cluster_hosted_zone_id"></a> [cluster_hosted_zone_id](#output_cluster_hosted_zone_id)                                                 | The Route53 Hosted Zone ID of the endpoint                                        |
| <a name="output_cluster_id"></a> [cluster_id](#output_cluster_id)                                                                                     | The RDS Cluster Identifier                                                        |
| <a name="output_cluster_instances"></a> [cluster_instances](#output_cluster_instances)                                                                | A map of cluster instances and their attributes                                   |
| <a name="output_cluster_master_password"></a> [cluster_master_password](#output_cluster_master_password)                                              | The database master password                                                      |
| <a name="output_cluster_master_username"></a> [cluster_master_username](#output_cluster_master_username)                                              | The database master username                                                      |
| <a name="output_cluster_members"></a> [cluster_members](#output_cluster_members)                                                                      | List of RDS Instances that are a part of this cluster                             |
| <a name="output_cluster_port"></a> [cluster_port](#output_cluster_port)                                                                               | The database port                                                                 |
| <a name="output_cluster_reader_endpoint"></a> [cluster_reader_endpoint](#output_cluster_reader_endpoint)                                              | A read-only endpoint for the cluster, automatically load-balanced across replicas |
| <a name="output_cluster_resource_id"></a> [cluster_resource_id](#output_cluster_resource_id)                                                          | The RDS Cluster Resource ID                                                       |
| <a name="output_cluster_role_associations"></a> [cluster_role_associations](#output_cluster_role_associations)                                        | A map of IAM roles associated with the cluster and their attributes               |
| <a name="output_db_cluster_parameter_group_arn"></a> [db_cluster_parameter_group_arn](#output_db_cluster_parameter_group_arn)                         | The ARN of the DB cluster parameter group created                                 |
| <a name="output_db_cluster_parameter_group_id"></a> [db_cluster_parameter_group_id](#output_db_cluster_parameter_group_id)                            | The ID of the DB cluster parameter group created                                  |
| <a name="output_db_parameter_group_arn"></a> [db_parameter_group_arn](#output_db_parameter_group_arn)                                                 | The ARN of the DB parameter group created                                         |
| <a name="output_db_parameter_group_id"></a> [db_parameter_group_id](#output_db_parameter_group_id)                                                    | The ID of the DB parameter group created                                          |
| <a name="output_db_subnet_group_name"></a> [db_subnet_group_name](#output_db_subnet_group_name)                                                       | The db subnet group name                                                          |
| <a name="output_enhanced_monitoring_iam_role_arn"></a> [enhanced_monitoring_iam_role_arn](#output_enhanced_monitoring_iam_role_arn)                   | The Amazon Resource Name (ARN) specifying the enhanced monitoring role            |
| <a name="output_enhanced_monitoring_iam_role_name"></a> [enhanced_monitoring_iam_role_name](#output_enhanced_monitoring_iam_role_name)                | The name of the enhanced monitoring role                                          |
| <a name="output_enhanced_monitoring_iam_role_unique_id"></a> [enhanced_monitoring_iam_role_unique_id](#output_enhanced_monitoring_iam_role_unique_id) | Stable and unique string identifying the enhanced monitoring role                 |
| <a name="output_security_group_id"></a> [security_group_id](#output_security_group_id)                                                                | The security group ID of the cluster                                              |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->

## Authors

Module is maintained by [Anton Babenko](https://github.com/antonbabenko) with help from [these awesome contributors](https://github.com/terraform-aws-modules/terraform-aws-rds-aurora/graphs/contributors).

## License

Apache 2 Licensed. See [LICENSE](https://github.com/terraform-aws-modules/terraform-aws-rds-aurora/tree/master/LICENSE) for full details.

## Additional information for users from Russia and Belarus

- Russia has [illegally annexed Crimea in 2014](https://en.wikipedia.org/wiki/Annexation_of_Crimea_by_the_Russian_Federation) and [brought the war in Donbas](https://en.wikipedia.org/wiki/War_in_Donbas) followed by [full-scale invasion of Ukraine in 2022](https://en.wikipedia.org/wiki/2022_Russian_invasion_of_Ukraine).
- Russia has brought sorrow and devastations to millions of Ukrainians, killed hundreds of innocent people, damaged thousands of buildings, and forced several million people to flee.
- [Putin khuylo!](https://en.wikipedia.org/wiki/Putin_khuylo!)
