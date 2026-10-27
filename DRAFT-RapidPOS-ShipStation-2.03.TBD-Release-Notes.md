# Rapid POS ShipStation Connector v2.03.TBD Release Notes

_Release Date: October 28, 2026_

---

## New Functionality

### Improved Name Handling for Sold To Name and Ship To Name

When the **Billing Name** or **Shipping Name** on a release ticket is blank, the connector now uses the first and last name from that address (if present) before falling back to the customer name. 

- The **Sold To Name** in ShipStation is now filled in using the first available value in this order: **Billing Name**, billing **First Name** and **Last Name**, customer name, and then customer first and last name.
- The **Ship To Name** follows the same order, starting with the **Shipping Name** and then the shipping **First Name** and **Last Name**, customer name, and then customer first and last name.
- If only a first name or only a last name is entered, that name is used rather than skipping ahead to the customer name.
- If no name is available at any level, the name is now sent to ShipStation as an empty value rather than as a single space.

For a full list of fields sent to ShipStation, see the ShipStation Connector README.

### Dedicated SQL Login for the Connector

The connector now connects to the Counterpoint database with its own dedicated SQL Login, instead of sharing the main SQL Login used by Counterpoint. Because the connector's database activity can now be identified separately from Counterpoint's, Rapid can limit how much of the SQL Server's CPU the connector uses, so that connector activity does not slow down Counterpoint for store users.

- The SQL Login is created automatically during the connector upgrade. 
- The new login receives the same server and database permissions as the existing Counterpoint SQL Login, so the connector continues to work the same way as before.
- A unique, strong password is generated for the login and is stored only in encrypted form in the connector's configuration. A new password is generated each time the connector is upgraded.
- Do not change the password for this SQL Login or remove the login. The connector depends on it, and its password is managed automatically by the upgrade process.

---

## Bug Fixes and Performance Enhancements

### Restored Tracking Number Import from ShipStation

Resolved an issue that could prevent tracking numbers from being imported from ShipStation into Counterpoint on some installations. Release tickets continued to send to ShipStation, but tracking numbers were not returned to Counterpoint.

- Tracking numbers import on the configured schedule (every 15 minutes by default).
- The same issue stopped the **Tracking Last Get Date (UTC)** field in the connector configuration from updating. This field shows the last time tracking information was imported, and it now updates each time the import runs.

### Restored Automatic Updates to the ShipStation Carriers Table

Resolved an issue that could prevent new carriers from being added to the **ShipStation Carriers** table in Counterpoint on some installations. This table links ShipStation carriers to Counterpoint ship-via codes, so a missing carrier means imported tracking numbers cannot update the ship-via code on the release ticket.

- When a new carrier is added in ShipStation, a matching entry is created in the **ShipStation Carriers** table automatically. The ship-via code on the new entry is left blank so it can be assigned in Counterpoint, if desired.

### ShipStation ID Saved Immediately for Instantly Sent Release Tickets

When a release ticket is sent to ShipStation as soon as it is completed, the ShipStation ID is now saved to the release ticket right away. Previously, the ID was not saved until the next scheduled run of the connector, so the release ticket briefly appeared as if it had not been sent.

- The ShipStation ID links the release ticket in Counterpoint to the matching order in ShipStation, and it is used when tracking numbers are imported.

### Automatic Recovery After a Server or SQL Server Restart

Resolved an issue where the connector could stop running after the server was rebooted or the SQL Server service was restarted, and would stay stopped until someone restarted the connector's Windows Service manually. 

- When the connector starts, it now waits up to 20 seconds for SQL Server to become available. This covers the common case where the connector starts before SQL Server has finished starting after a reboot.
- If SQL Server is still unavailable after 20 seconds, the connector now reports the startup as failed, so that Windows can restart the service automatically.
- If the connection to SQL Server is lost while the connector is running, the connector now waits between retries, starting at 5 seconds and increasing up to 5 minutes, instead of retrying continuously. This keeps CPU usage low during an outage, and normal processing resumes when SQL Server is available again.
- Connection problems, and the time the connection is restored, are now recorded in the connector log.
