---
name: weekly-campaign-review
description: Run a weekly review of the user's GTM Bud campaigns. Use when the user asks for a weekly review, a campaign check-in, how their campaigns perform, or what to do next in GTM Bud.
---

# Weekly campaign review

Give the user one short report on their GTM Bud campaigns. The report has four parts: campaign results with a grade for the acceptance rate, replies that need an answer, sending account problems, and 3 next actions.

Use only the GTM Bud tools for data. Make no changes during the review. Offer each change as a next action, and do it only when the user asks.

## Steps

1. **Find the active campaigns.** Call `list_campaigns`. Follow `nextCursor` until no cursor is left. Keep the campaigns with status `active`. If no campaign is active, say so, list the paused and draft campaigns by name, and go to step 4.
2. **Get the results.** Call `get_campaign_results` for each active campaign. Grade the acceptance rate of each campaign with the rules below. The step is done when each active campaign has a grade, or a note that gives the reason for no grade.
3. **Find the replies that need an answer.** Call `list_replies` for each active campaign with `needs_answer_only: true`. Follow `nextCursor`. The list gives no reply text. Call `get_conversation` for each reply that the report will show, and read the last message of the lead. The step is done when you have the full list for each active campaign and the text of each reply that you show.
4. **Check the sending accounts.** Call `list_sending_accounts`. A problem is an account with a `connectionStatus` that is not connected, or an account with `paused: true`. Also note an account with `salesNavigator: false`: it is an account without Sales Navigator. GTM Bud finds more leads for an account with Sales Navigator.
5. **Write the report** in the format below.
6. **Suggest one playbook improvement** after the report. Call `get_reply_playbooks`. Compare the playbooks with what the replies of the week showed, such as a question that leads often ask. Show one change as the exact new text. Call `update_reply_playbook` only after the user approves that text.

## Results data

Read each number from `results` of `get_campaign_results`.

| Number | Where it is |
| --- | --- |
| Connection requests sent | `connectionRequestAcceptanceRate.denominator.count` |
| Connection requests accepted | `connectionRequestAcceptanceRate.numerator.count` |
| Acceptance rate | `connectionRequestAcceptanceRate.percent` |
| Acceptance target | `connectionRequestAcceptanceRate.target`: `minimumPercent`, `okPercent`, `greatPercent` |
| Leads reached | `leadsReached.count`, and `leadsReached.byChannel` for each channel |
| Conversations | `conversations.count` |
| Leads that replied | `totalReplyRate.numerator.count` |
| Replies by label | The `numerator.count` of `interestedReplyRate`, `warmReplyRate`, `notInterestedReplyRate` and `unclassifiedReplyRate` |

Read these limits of the data before you grade:

- The numbers are totals since the campaign launched, not numbers for the last 7 days. Write "since launch" next to each rate. If you have the report of last week in this chat, compare with it to show the change.
- `asOf` at the top level of the tool result is the time of the numbers. Give it in the report. If the result has `lastRefreshFailed: true`, say that the numbers are from the time in `asOf`.
- `results` is null while GTM Bud prepares the first numbers of a campaign. Write "no results yet" for that campaign.
- Each reply rate uses Conversations as the denominator, not leads reached. A lead counts one time, with its strongest reply label.
- The reply labels are not split by channel. The results have no count of InMails sent and no count of meetings.

## Acceptance grade

Grade only the connection request acceptance rate. Use the levels in `connectionRequestAcceptanceRate.target`.

| Acceptance rate | Grade |
| --- | --- |
| Under `minimumPercent` | Below target |
| From `minimumPercent` to under `okPercent` | Minimum |
| From `okPercent` to under `greatPercent` | OK |
| `greatPercent` or more | Great |

- Give no grade when `status` is `awaiting_first_snapshot` or `percent` is null. Write "no acceptance rate yet". In this state the result has no counts and no target for the acceptance rate. Give `leadsReached.byChannel.connectionRequest` as the leads that got a connection request.
- Write "not enough data" when the connection requests sent are under 20.
- Give no grade to a reply rate. GTM Bud gives no target for the reply rates. Report the counts.

## Report format

Write in short, plain sentences. Use the campaign names, not IDs. Add the `campaignUrl` link of each campaign when the tool gives one.

1. **Campaigns.** One line for each active campaign: name, connection requests sent, acceptance rate and grade, conversations, leads that replied, and the replies by label (Interested, Warm, Not interested, not labeled). Put the campaign with the lowest grade first, and the campaigns with no grade last.
2. **Replies that need an answer.** Count for each campaign, then the lead name and one line on what the lead said, newest first. Show at most 10 and give the count of the others. Add the `inboxUrl` link.
3. **Sending accounts.** One line for each problem account: name, problem, and the fix. For a disconnected account, add the `connectAccountUrl` link. Write "No problems" when the list is empty.
4. **Next actions.** Exactly 3 actions, most important first. Each action names one campaign, reply or account, the result it changes, and the GTM Bud tool that does it when one exists. Use this order of priority:
   - A disconnected or paused sending account. It sends no messages until it is fixed. The tools do not show which campaign uses an account, so tell the user to check the campaign senders on gtmbud.com.
   - Replies that need an answer. Use one action for all replies of one campaign, and start with the oldest reply. Offer to write a reply for each. Read it with `get_conversation`, follow its `replyGuidance`, show your text with `review_reply`, then use `send_reply` only after the user approves the text.
   - A campaign below the acceptance minimum. Suggest a tighter audience: check its leads with `list_campaign_leads`, and remove poor fits with `remove_lead`.
   - A campaign with acceptance at or above the minimum and few Interested replies for its conversations. The message is a possible cause. Show the two counts, and suggest a message change in the GTM Bud app.
   - A campaign that sends fewer connection requests than the user expects. Check the sending accounts and the number of assigned leads.
