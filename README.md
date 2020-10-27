# lightrail
This document is your guide on how to care for Lightrail after all the elves have moved on to the Undying Lands.

## The 50 Foot View

## Monitoring for Trouble

Developers are notified of emergencies through [PagerDuty](https://giftbit.pagerduty.com).  The only alarms configured are 5xx spikes and latency spikes.

Developers are notified of system errors through [Sentry](https://sentry.io/organizations/giftbit/issues/).  The `react-frontend` project can be a bit noisy but the backend projects rarely throw errors.

For a picture of general system health see the DataDog dashboards.  [Lightrail Left and](https://app.datadoghq.com/dashboard/qe9-ueb-qh3/lightrail-left) is focused on business metrics and [Lightrail Right](https://app.datadoghq.com/dashboard/w4b-e9j-3wy/lightrail-right) is focused on technical metrics.

## Troubleshooting an Outage

Step one is determining where the problem is.  The Datadog dashboard [Lightrail Error Responses](https://app.datadoghq.com/dashboard/mbb-7vd-zjb/lightrail-error-responses) is useful here.  You can also look in Sentry.

Occasionally ApiGateway or Lambda has a mini-outage.  This will look like 5xx errors in ApiGateway and the Lambda fails to invoke.  These errors fix themselves.

If the bug is in the code there will be log output for it.  The logs will be stored in CloudWatch under the group name `/aws/lambda/<env>-<service>-<function>-<rand>` where `<env>` is the environment name (production), `<service>` is the service name (eg: Rothschild), `<function>` is the name of a Lambda function in the service (eg: RestFunction) and `<rand>` is some random characters set when the service was first deployed.  Search the log group for a string like `"status=500"`.

If the RDS instance is overtaxed the RDS instance Monitoring tab provides detailed metrics.  Performance Insights is enabled on the production RDS instance for a view into what SQL is problematic.

## Revoking a Customer's Access

## Deploying a Code Change

## Shutting it all Down
