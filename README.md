# MSDR
A reminder framework built in Marketo.

Crafted specifically for sending alert emails to SDRs when it's time to call a lead.

## Why MSDR?
In some sales processes, it's good practice to have a human talk to inbound leads before passing them to sales reps. The person doing this is called the SDR ("Sales Development Role") and responds to internal lead alerts by contacting the customer.

Marketo's best practices suggest attempting to reach the lead six times based on a schedule of increasing intervals--e.g., within 5 minutes, 30 minutes, 4 hours, 3 days, 7 days, and 14 days.

The MSDR framework makes it possible for anyone with Marketo to employ this sales qualification technique.

### Features
* Reminders are sent according to intervals, each of which is set independently
* Manually override schedule with a specific date/time for following up with a lead (called "lead snooze")
* Automatically queue reminders if SDRs are all unavailable during otherwise normal hours
* Automatically queue reminders outside of normal business hours
* One-click options for recycling, rejecting, or qualifying a lead, all of which can turn off the SDR phase and trigger lead lifecycle campaigns elsewhere

## Basics
MSDR consists of:

* Smart campaigns to run the program
* A new channel ("SDR")
* A few new lead field to trigger next steps (snooze until, reject, recycle, qualify)
* A custom lead layout for conducting the SDR call
* An additional smart campaign to sync the Marketo Lead ID field (used embedding a link to the Marketo lead view within the alert email)
* Two flat lists and one smart list