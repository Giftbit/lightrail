# lightrail
This document is your guide on how to care for Lightrail after all the elves have sailed to the Undying Lands.

## The 50 Foot View

Lightrail is a primarily a REST API defined in [lightrail-cloudformation-infrastructure](https://github.com/Giftbit/lightrail-cloudformation-infrastructure/blob/master/modules/cloudfront-api-distribution-template.yaml).  The REST API is implemented by microservices that mostly don't talk to each other, but when they do they do it through the REST API or the [Lightail event topic](https://github.com/Giftbit/lightrail-cloudformation-infrastructure/blob/master/lightrail-stack.yaml#L194).

The primary microservices are [Rothschild](https://github.com/Giftbit/internal-rothschild) covering /v2/currencies, /v2/contacts, /v2/programs, /v2/values; [Edhi](https://github.com/Giftbit/internal-edhi) covering /v2/account, /v2/user; [KVS](https://github.com/Giftbit/internal-kvs) covering /v1/storage; and [Gutenberg](https://github.com/Giftbit/internal-gutenberg/) covering /v2/webhooks.  This is all defined in [lightrail-cloudformation-infrastructure's cloudfront-api-distribution-template.yaml](https://github.com/Giftbit/lightrail-cloudformation-infrastructure/blob/master/modules/cloudfront-api-distribution-template.yaml#L102).

Philisophically the microservices could each have been implemented in different ways, but in practice they ended up the same for simplicity.  Each microservice (links are to Rothschild as an example) is deployed by a CodePipeline created by [lightrail-cloudformation-infrastructure](https://github.com/Giftbit/lightrail-cloudformation-infrastructure/blob/master/modules/rothschild.yaml#L122) that points to a specific commit of [infrastructure/ci.yaml](https://github.com/Giftbit/internal-rothschild/blob/staging/infrastructure/ci.yaml).  The CodePipeline deploys [infrastrcutrue/sam.yaml](https://github.com/Giftbit/internal-rothschild/blob/staging/infrastructure/sam.yaml) which includes resources like databases, and one or more Lambdas.  The Lamba source code is in [src/lambdas/](https://github.com/Giftbit/internal-rothschild/tree/staging/src/lambdas).  REST Lambdas handle multiple HTTP request paths, routing them with [Cassava](https://github.com/Giftbit/cassava).  Other Lambdas may be triggered on a schedule, by an SQS queue or an SNS topic.

Authentication is done with JWTs signed with a secret stored in the lightrailsecureconfig S3 bucket.  All JWTs with valid signatures are 100% trusted with no extra steps such as checking that the userId exists.  The exception is API keys blocklisted by the WAF WebACL.

The system is deployed in three accounts: dev, staging and production.  Dev is for development and may have changes being actively developed.  Staging is for a dry run of deployment and ironing out any issues before going live.  Production is where the money ~is~ was earned.  Being a serverless architecture the dev and staging accounts are fairly cheap to maintain and I would keep those turned on as long as production is on just in case.

## Monitoring for Trouble

Developers are notified of emergencies through [PagerDuty](https://giftbit.pagerduty.com).  The only alarms configured are 5xx spikes and latency spikes.

Developers are notified of system errors through [Sentry](https://sentry.io/organizations/giftbit/issues/).  The `react-frontend` project can be a bit noisy but the backend projects rarely throw errors.

For a picture of general system health see the DataDog dashboards.  [Lightrail Left](https://app.datadoghq.com/dashboard/qe9-ueb-qh3/lightrail-left) is focused on business metrics and [Lightrail Right](https://app.datadoghq.com/dashboard/w4b-e9j-3wy/lightrail-right) is focused on technical metrics.

## Troubleshooting an Outage

Step one is determining where the problem is.  The Datadog dashboard [Lightrail Error Responses](https://app.datadoghq.com/dashboard/mbb-7vd-zjb/lightrail-error-responses) is useful here.  You can also look in Sentry.

Occasionally ApiGateway or Lambda has a mini-outage.  This will look like 5xx errors in ApiGateway and the Lambda fails to invoke.  These errors fix themselves.

If the bug is in the code there will be log output for it.  The logs will be stored in CloudWatch under the group name `/aws/lambda/<env>-<service>-<function>-<rand>` where `<env>` is the environment name (production), `<service>` is the service name (eg: Rothschild), `<function>` is the name of a Lambda function in the service (eg: RestFunction) and `<rand>` is some random characters set when the service was first deployed.  Search the log group for a string like `"status=500"`.

If the RDS instance is overtaxed the RDS instance Monitoring tab provides detailed metrics.  Performance Insights is enabled on the production RDS instance for a view into what SQL is problematic.

## Revoking a Customer's Access

I'm going to write a script that does this.

## Deploying a Backend Change

The backend is built on several microservices.  Primarily they are [Rothschild](https://github.com/Giftbit/internal-rothschild), [Edhi](https://github.com/Giftbit/internal-edhi), [KVS](https://github.com/Giftbit/internal-kvs) and [Gutenberg](https://github.com/Giftbit/internal-gutenberg/).  Each of these services is deployed by CodePipeline.  This includes both code and infrastructure (via CloudFormation).  The staging account is deployed from the staging branch and the production account is deployed from the master branch.

The development flow is:
- develop locally against unit tests (`npm run test`)
- deploy to dev with `./dev deploy` as necessary
- open a GitHub PR to the staging branch
- someone else approves the PR
- merge the PR which automatically starts the staging CodePipeline
- approve the CodePipeline changes in the staging account and watch for it to complete which automatically creates a PR from staging to master
- approve and merge the GitHub PR to master which automatically starts the production CodePipeline
- approve the CodePipeline changes in the production account and watch for it to complete

If you PR directly to master you can shortcut some steps above but I really don't recommend it.  Don't be a cowboy.

Unifying infrastructure not in one of the above projects is defined in [lightrail-cloudformation-infrastructure](https://github.com/Giftbit/lightrail-cloudformation-infrastructure/).  This includes IAM, CloudFront, S3 buckets, KMS and CodePipelines.  Unlike the above projects that have a CodePipeline in each environment, lightrail-cloudformation-infrastructure has a single CodePipeline called LightrailInfrastructureCI that lives in production.  This CodePipeline has stages and permissions to deploy across AWS accounts.

The only infrastructure not managed by any of the above is infrastructure that has to live in the us-east-1 region: domain names, certificates, Lambda@Edge and WAF WebACL.  *I strongly recommend you manage all changes possible through CloudFormation templates deployed by CodePipeline to keep the system consistent.*  That is unless you're closing the account.  Then you can go wreck house.

## Deploying a Webapp Change

See [lightrail-webapp](https://github.com/Giftbit/lightrail-webapp/)

## Updating the Static Site

See [lightrail-static](https://github.com/Giftbit/lightrail-static)

## Shutting it all Down
1. Take down the website by deleting the CloudFront distributions.
2. Backup all production data you may need to refer to later.
    1. Connect to the RDS instance through the bastion host as outlined in the [Rothschild](https://github.com/Giftbit/internal-rothschild) readme.  Connect to the database with MySQL Workbench and use the [data export wizard](https://dev.mysql.com/doc/workbench/en/wb-admin-export-import-management.html) to export all the data.  This data contains personally identifying information so store it somewhere secure.
    2. Download the entire [Edhi](https://github.com/Giftbit/internal-edhi) database with the script `export`.  This data contains personally identifying information so store it somewhere secure.
3. [Transfer the domains](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-transfer-to-route-53.html) out of the account.  `lightrail.com` at least is worth a little bit of money.
4. Initiate [close account](https://aws.amazon.com/premiumsupport/knowledge-center/close-aws-account/) on all 3 accounts (dev, staging, production).
