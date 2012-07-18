# MSDR
A reminder framework built in Marketo. Crafted specifically for sending alert emails to SDRs when it's time to call a lead.

MSDR is currently in use at [Knewton](http://www.knewton.com/).

**Why MSDR?**
In some sales processes, it's good practice to have a human talk to inbound leads before passing them to sales reps. The person doing this is called the SDR ("Sales Development Role") and responds to internal lead alerts by contacting the customer.

Marketo's best practices suggest attempting to reach the lead six times based on a schedule of increasing intervals--e.g., within 5 minutes, 30 minutes, 4 hours, 3 days, 7 days, and 14 days.

The MSDR framework makes it possible for anyone with Marketo to employ this sales qualification technique.

**Features**
* Reminders are sent according to intervals, each of which is set independently
* Manually override schedule with a specific date/time for following up with a lead (called "lead snooze")
* Automatically queue reminders if SDRs are all unavailable during otherwise normal hours
* Automatically queue reminders outside of normal business hours
* One-click options for recycling, rejecting, or qualifying a lead, all of which can turn off the SDR phase and trigger lead lifecycle campaigns elsewhere

**Summary**
At a minimum, MSDR consists of:

* A new channel ("SDR") with custom progression statuses
* A bunch of smart campaigns to run the program
* Two flat lists, one smart list
* A few new lead fields in Marketo for triggering SDR program changes
* A custom lead layout for conducting the SDR call
* An additional smart campaign to sync the Marketo Lead ID field

Optional additions:

* An exclusion smart list just for this program
* Lead fields for collecting answers to your questions with leads

## Setup

### Prerequisites

 1. Create the SDR Channel
It should [look like this](https://github.com/superstrong/MSDR/blob/master/Admin%20Changes/sdr-channel-setup.png).

 2. Create the lists
* The [two flat lists](https://github.com/superstrong/MSDR/blob/master/DB%20Changes/lead-database-lists.png) are empty--they'll be used by interruption campaigns to queue everyone affected and then trigger the alert when the interruption is over
* The [smart list](https://github.com/superstrong/MSDR/blob/master/DB%20Changes/smart-list-sdr-offline.png) captures everyone still active in the SDR program. The SDR(s) use this campaign to trigger manually the campaign to take the SDR offline until a specific dateTime. This is only used when all SDRs are unavailable during an otherwise normal business time and wish to queue incoming alerts.
* Optionally, create another smart list that serves as an exclusion list for the program. Creating a smart list just for SDR Exclusions--which also contains your Global Exclusion List--gives you the flexibility to add more restrictive filters to prevent you from accidentally calling anyone who shouldn't be called.

 3. Create the new Marketo database lead fields
There are [four lead fields](https://github.com/superstrong/MSDR/blob/master/Admin%20Changes/lead-custom-fields-with-layout.png) needed to trigger next steps and [one more lead field](https://github.com/superstrong/MSDR/blob/master/Additional%20Campaigns/marketo-id-sync.png) needed to provide a link in the alert email directly to the editable Market lead view.
* SDR Manual Snooze Until At - dateTime
* SDR Reject Lead - checkbox
* SDR Recycle Lead - checkbox
* SDR Qualify Lead - checkbox
* Marketo Lead ID - string

Optionally, create additional lead fields to collect answers to the questions your SDR(s) will ask on the phone. These can be synchronized with Salesforce and/or inserted into an email to your sales team once a lead is qualified.

 4. Create the SDR Program and folders within it
To keep it simple, replicate [the structure you see here](https://github.com/superstrong/MSDR/blob/master/program-overview.png).

 5. Create [the alert email](https://github.com/superstrong/MSDR/blob/master/_%20Program%20Layout/Emails/alert-email.png), which contains the [direct link to the lead view](https://github.com/superstrong/MSDR/blob/master/_%20Program%20Layout/Emails/lead-detail-URL.png). Note that your subdomain may vary--use whatever you see when you log in to Marketo.

### Required smart campaigns
Following [the structure detailed here](https://github.com/superstrong/MSDR/tree/master/_%20Program%20Layout), create all the smart campaigns.

Most campaigns use "Campaign is Requested" (Marketo Flow Action) as the trigger. The icon for this type of smart campaign is a light bulb with a green arrow on top. When building your smart campaigns, you can even start by having all campaigns use this trigger; this will allow you to activate everything without fear that you will accidentally start anything in motion.

You'll need to activate every campaign as you go in order to make it visible to other campaigns. In the "Schedule" section, choose "Run flow every time" and activate. You can change this frequency later if you like.