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
There are [four lead fields](https://github.com/superstrong/MSDR/blob/master/Admin%20Changes/lead-custom-fields-with-layout.png) needed to trigger next steps, two more to handle other interruptions, and [one more lead field](https://github.com/superstrong/MSDR/blob/master/Additional%20Campaigns/marketo-id-sync.png) needed to provide a link in the alert email directly to the editable Market lead view.
* SDR Manual Snooze Until At - dateTime
* SDR Offline Until At - dateTime
* SDR Window Closed - checkbox
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

## Explanation
The SDR program is broken into three sections: Campaign Logic, Emails, and Progression Statuses.

The Campaign Logic has three sections:

 1. Run Program - keeps leads in a wait step for pre-defined amounts of time--e.g., 30 minutes or 4 hours
 2. Alert SDR - looks for interruptions and sends the alert email
 3. Interruptions - handles deviations from the pre-defined schedule

Leads bubble down--and back up--through the sections. For example, when a pre-defined wait step has passed, a lead is moved to the "Alert SDR" section. If it's now after business hours, a "Window Closed" interruption will take over, keeping the lead in a wait step, then sending it back up to the "Alert SDR" section when the wait step completes.  Once the alert has successfully been sent, the progression status and program status changes, and the lead moves back up to the "Run Program" section to wait in the next pre-defined wait step.

Once started, the program runs itself without intervention by the SDR, meaning it operates under the assumption that an SDR will attempt a phone call whenever she receives an alert and that she will not reach the lead. The program only stops when it runs out of steps (currently 6 call attempts) or is stopped via a smart campaign in the "Leave Flow" folder.

### Nuances

Because the new "SDR" lead fields (reject, recycle, etc.) will trigger changes _immediately_, the campaigns listening for those changes have a 1-minute wait step built into them. This gives you one minute to undo your mistake. If after one minute the value has changed back--a checkbox unchecked or a dateTime returned to null--the smart campaign will stop itself.

Only five smart campaigns do not use "Campaign is requested" as triggers:

 1. "SDR Go Offline" - the SDR must request this manually. (While viewing the "SDR Offline" smart list members, the SDR should "select all", then request the "SDR Go Offline" campaign.)
 2. "SDR Trigger Lead Snooze" - looks for changes to "SDR Manual Snooze Until At" where the new value is not NULL
 3. "SDR Trigger Lead Snooze - Undo" - looks for changes to the "SDR Manual Snooze Unti At" where the new value is NULL
 4. "SDR Window Close" - M-F at 6pm, grabs all leads active in the program
 5. "SDR Window Open" - M-F at 10am, grabs all leads active in the program