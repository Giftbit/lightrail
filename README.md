# lightrail
This document is your guide on how to care for Lightrail after all the elves have moved on to the Undying Lands.

## The 50 Foot View

## Monitoring for Trouble

Developers are notified of emergencies through [PagerDuty](https://giftbit.pagerduty.com).  The only alarms configured are 5xx spikes and latency spikes.

Developers are notified of system errors through [Sentry](https://sentry.io/organizations/giftbit/issues/).  The `react-frontend` project can be a bit noisy but the backend projects rarely throw errors.

For a picture of general system health see the DataDog dashboards.  [Lightrail Left and](https://app.datadoghq.com/dashboard/qe9-ueb-qh3/lightrail-left) is focused on business metrics and [Lightrail Right](https://app.datadoghq.com/dashboard/w4b-e9j-3wy/lightrail-right) is focused on technical metrics.

## Troubleshooting an Outage

## Revoking a Customer's Access

## Deploying a Code Change

## Shutting it all Down
