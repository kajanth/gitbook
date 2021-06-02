# Naming

## [AWS](README.md) / Naming

_Naming_ is one of the [hard problems in computing](https://martinfowler.com/bliki/TwoHardThings.html). Naming instances in a shared name space, such as AWS, is especially rife with problems. Often a company uses a single AWS account across multiple projects and so the projects have to negotiate how to share the namespace for resources. There are a number of different axes of belonging we might want to include in the name: to which project does a resource belong, to which environment \(dev, prod, staging\), which function within the environment. Then there are entities which exist only in terms of their relationship to other objects, e.g. roles associated with a particular lambda function.

Things are further complicated in AWS because there are different uniqueness constraints in play:

| Uniqueness constraint | Example resources |
| :--- | :--- |
| Global \(the entirety of AWS\) | S3 buckets |
| Per account | IAM resources \(users, groups, roles, policies, …\) |
| Per region | Most everything else \(ALB, ECS, RDS, …\) |

As with all naming schemes \(and other stylistic things such as casing and comments\) where the client already has a functional naming scheme we should follow that - there are more important issues to deal with. However, for our own work and for projects where we are setting the standard tend to use the following.

* Globally unique
  * S3 Buckets
* Unique per account
  * IAM resources
    * IAM policies
* Unique per region

### Globally unique

#### S3 Buckets

_${account.alias}-${application.name}\[-${environment}\]\[-${region}\]_ - these names begin with a consistent account/usage prefix as they are globally scoped across all of AWS.

* _$account-alias_ - is a prefix for the account, e.g. "truss", "client-name"
* _$application-name_ - is application for which the resource is created, e.g. "aws-logs", "webserver", "terraform-state"
* _$environment_ - can be used to distinguish different versions of the resource/app that occur during the development lifecycle, e.g. `dev`, `perf_test`, `staging` and `prod`uction
* _$region_ - when an app can or will be distributed across AWS regions with distinct instances in each region, this postfix distinguishes between them

e.g.

* truss-aws-logs-us-east-1

### Unique per account

#### IAM resources

IAM resource names are account unique \(i.e., visible across all regions\), e.g. for roles:

_${service/realm}\[-${role}\]-${project/application}-${environment}_ - is the general form

* _$service/realm_ - to what does this role pertain, e.g. "ecs", "lamda" or "circleci".
* _$role_ - where there may be multiple roles associated with a service, this can be used as a way of disambiguating, e.g "task-execution" or "rds-snapshot-cleaner"
* _$project/application_ - where there may be multiple applications/projects being managed with independent deploy cycles, this is used, e.g. "webapp", "honeycomb"
* _$environment_ - can be used to distinguish different versions of the resource/app that occur during the development lifecycle, e.g. `dev`, `perf_test`, `staging` and `prod`uction

e.g.

* lambda-rds-snapshot-cleaner-app-experimental
* ecs-task-role-app-client-tls-experimental

**IAM policies**

Where the details of an IAM role are in an associated policy \(usually the case\) they are named after the associated role, but with `-policy` appended, e.g.

* lambda-rds-snapshot-cleaner-app-experimental-policy
* ecs-task-role-app-client-tls-experimental-policy

### Unique per region

Where the name is scoped by the resource type and the region, e.g. lambda functions, then it is enough to give a meaningful name and qualify by environment. If the purpose is common, e.g. rds-log-cleaner, it may need an application.

_${purpose}\[-${application}\]-${environment}_ - is the general form

* _$purpose_ - a simple name describing the role/purpose of the resource, e.g. "slack-pivotal-bot", "webserver", "rds-log-cleaner"
* _$application_ - if needed, disambiguates between a similar purpose across applications, e.g. "webapp" vs "honeycomb"
* _$environment_ - can be used to distinguish different versions of the resource/app that occur during the development lifecycle, e.g. `dev`, `perf_test`, `staging` and `prod`uction

e.g.

* slack-pivotal-tracker-bot-test
* rds-log-cleaner-webapp-prod

