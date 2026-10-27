# Rapid POS ShipStation Connector v2.03.TBD Release Notes

_Release Date: October 28, 2026_

---

## New Functionality

### Improved Name Handling for Sold To Name and Ship To Name

When the **Billing Name** or **Shipping Name** on a release ticket is blank, the connector now uses the first and last name from that address before falling back to the customer name. This helps orders in ShipStation show the name of the person the order is for, rather than a general customer account name.

- The **Sold To Name** in ShipStation is now filled in using the first available value in this order: **Billing Name**, billing **First Name** and **Last Name**, customer name, and then customer first and last name.
- The **Ship To Name** follows the same order, starting with the **Shipping Name** and then the shipping **First Name** and **Last Name**, customer name, and then customer first and last name.
- If only a first name or only a last name is entered, that name is used rather than skipping ahead to the customer name.
- If no name is available at any level, the name is now sent to ShipStation as an empty value rather than as a single space.

For a full list of fields sent to ShipStation, see the [ShipStation Connector documentation]({{DOCUMENTATION_LINK}}).

---

## Bug Fixes and Performance Enhancements

### Restored Tracking Number Import from ShipStation

Resolved an issue that could prevent tracking numbers from being imported from ShipStation into Counterpoint on some installations. Release tickets continued to send to ShipStation, but tracking numbers were not returned to Counterpoint.

- Tracking numbers and their carrier codes now import on the configured schedule (every 15 minutes by default).
- The same issue stopped the **Tracking Last Get Date (UTC)** field in the connector configuration from updating. This field shows the last time tracking information was imported, and it now updates each time the import runs.

### Restored Automatic Updates to the ShipStation Carriers Table

Resolved an issue that could prevent new carriers from being added to the **ShipStation Carriers** table in Counterpoint on some installations. This table links ShipStation carriers to Counterpoint ship-via codes, so a missing carrier means imported tracking numbers cannot update the ship-via code on the release ticket.

- When a new carrier is added in ShipStation, a matching entry is created in the **ShipStation Carriers** table automatically. The ship-via code on the new entry is left blank so it can be assigned in Counterpoint, if desired.
