# ShipStation Connector v2.04.05 Release Notes

_Release Date: November 10, 2026_

---

## Bug Fixes and Performance Enhancements

### Ticket Sending When SQL Server and Counterpoint Are on Separate Machines

In some installations, SQL Server and the Counterpoint application server run on separate machines. In these installations, completed tickets were no longer sent to ShipStation right away, because, due to a recent security enhancement, the connector can only open its secure database connection on the machine where it is installed. For these installations only, the connector now checks for completed tickets every few seconds and sends them to ShipStation, so tickets still reach ShipStation within seconds of being completed.

>**All other installations still send tickets to ShipStation when they are completed, with no change.**

- This change applies only to installations where SQL Server and Counterpoint are on separate machines.
  - Rapid turns it on for those installations by setting the **Fast Poll** pipeline library variable to Y. The connector then reads completed tickets from the `USER_VI_SHIP_STATION_HDR_ELIG` view instead of sending them at ticket completion.
- The connector starts by checking every 5 seconds, and Rapid can shorten this to as little as 2 seconds with the **OrderPollSeconds** setting.
  - In testing, a ticket reached ShipStation in about 4 seconds with a 5-second check and about 3 seconds with a 2-second check.
- These checks run separately from the scheduled catch-all sync, which continues to send transfers and any orders that may have been missed by these checks.
- The connector can send about 20 orders per minute (about 1,200 per hour). ShipStation sets this limit, not the check interval.
  - ShipStation allows 40 requests per minute for each API key.
  - Each order uses two requests: one to confirm the order is not already in ShipStation, which prevents duplicate orders, and one to send it.
  - Other connector jobs on the same API key share this limit, such as shipment status updates and transfers.
- When many tickets are completed at once, the connector sends them oldest first. If ShipStation's limit is reached, the connector pauses until the limit resets and then continues. For example, 30 tickets completed at the same time are all sent in about a minute and a half.
- If a ticket fails to send, the connector waits before trying that ticket again, starting at 30 seconds and increasing up to 30 minutes. This keeps one problem ticket from holding up the others.
- In the connector log, orders sent by these checks are labeled (Poll).

### ShipStation IDs for Transfers

When the connector sent a group of transfers to ShipStation and found that one of them already existed there, it could also give that transfer's ShipStation ID to the remaining unsent transfers in the group.

- Since those transfers were still sent to ShipStation as usual, and each one later received its own correct ShipStation ID in Counterpoint, this issue is unlikely to have caused any visible problems.
- The connector now gives the existing ShipStation ID only to the matching transfer, so every transfer keeps its correct ShipStation ID throughout the sync.
