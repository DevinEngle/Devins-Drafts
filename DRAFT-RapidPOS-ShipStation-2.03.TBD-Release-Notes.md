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

Resolved an issue that could prevent tracking information from being imported from ShipStation into Counterpoint on some installations. Release tickets continued to send to ShipStation, but tracking numbers were not returned to Counterpoint.

- The same issue prevented new carriers from being added to the **ShipStation Carriers** table and stopped the **Tracking Last Get Date (UTC)** field from updating in the connector configuration.
- With this fix, tracking numbers and carrier codes import on the configured schedule (every 15 minutes by default), and new ShipStation carriers are added to the **ShipStation Carriers** table automatically.
