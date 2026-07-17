# Rapid POS WooCommerce Connector v2.01.00 Release Notes

_Release Date: July 21, 2026_

---

## New Functionality

### FFL Cockpit Order Processing (`USE_FFL_COCKPIT` Config Setting)

Clients selling firearms online through the **FFL Cockpit** plugin for WooCommerce can now have the connector automatically identify drop-ship orders fulfilled through that plugin, rather than requiring every order line to be manually swapped to a specific item in Counterpoint.

FFL Cockpit is a WooCommerce plugin used by firearms retailers to manage compliance-related order fulfillment and vendor drop-shipping.

- A new configuration setting for **Use FFL Cockpit**, `USER_WOOCOMMERCE_CONFIG.USE_FFL_COCKPIT`, controls this behavior. It is a yes/no checkbox and defaults to **No**, meaning existing clients will see no change in behavior unless this setting is explicitly enabled.
- When `USE_FFL_COCKPIT = N` (default): the connector continues to use its standard order import logic and falling back to the `WOOCOMM_INTERIM_ITEM` placeholder for any line that doesn't have a match. This is unchanged from prior versions.
- When `USE_FFL_COCKPIT = Y`: during order import, the connector inspects the `_fflc_fulfilled_items` metadata array included on the WooCommerce order (populated by the FFL Cockpit plugin). For most order line identified as being drop-shipped from a vendor (excluding serialized firearm items that are being picked up in-store), the connector assigns the `DROP_SHIP_FULFILL` item number instead of relying on standard item matching.
  - This distinguishes lines that FFL Cockpit is already handling through its own fulfillment process from lines that still require standard handling in Counterpoint.
  - Regulated firearm items that must be entered into a store's bound book for acquisition and disposition tracking will not be auto-assigned `DROP_SHIP_FULFILL`. These continue to come through as `WOOCOMM_INTERIM_ITEM`, requiring the Counterpoint user to manually swap in the correct existing or newly created item, since serialized firearm items need to exist individually in Counterpoint for compliance recordkeeping. (Serialized firearms that are being drop-shipped to another FFL will use `DROP_SHIP_FULFILL` since those do not need to be entered into the Counterpoint store's inventory.)

**Note:** Clients must have `DROP_SHIP_FULFILL` set up as a non-inventory item in Counterpoint prior to enabling this setting.

### Null Value Support for Custom Field Mapping Table

Clients can now clear a previously synced value in WooCommerce by clearing the corresponding field in Counterpoint — a capability that was not previously available for fields managed through the **WooCommerce Custom Field Mapping** table.

- Previously, the connector was designed to skip sending null values to WooCommerce entirely, in order to avoid overwriting other nested product data. This meant that once a value was set for a mapped Metafield, Product Property, or Attribute, it could not be unset from Counterpoint.
- The connector now sends null values specifically for fields tied to the custom field mapping table, allowing these values to be properly cleared in WooCommerce when the source field in Counterpoint is cleared.
- This has been confirmed working for **Metafield** and **Product Property** field types. For **Attribute** field types, clearing the value now removes the corresponding entry from that attribute's `options` array in WooCommerce.
- **Known limitation:** There is a known issue where the **Product Property** field type does not correctly null out in all cases, due to a constraint in the WooCommerce library used by the connector. A fix for this is planned for a future release.

---

## Bug Fixes and Performance Enhancements

### Faster Account Creation During Connector Install

Adjusted the connector's install script so that it can automatically create the necessary General Ledger account numbers during installation, when a client is using Rapid POS's default account setup — speeding up the install process for our install team.

- If the client uses profit centers, the script creates the required Main accounts (if not already present), then creates Posting accounts only for the accounts that were newly created — avoiding unintended changes to a client's existing account configuration.
- If the client does not use profit centers, the script creates the required accounts directly as both Main and Posting accounts, if not already present.

### Deprecated Unused Configuration Fields

Removed five configuration fields from the connector's install process that were found to be undocumented and unused by any current connector logic. Their presence in configuration files was creating confusion for both clients and installers.

The following fields have been deprecated and are no longer created or referenced during install:

- `SyncCustomers`
- `OverrideExistingSalePrice`
- `UseEmailAsShipToEmail`
- `ImportProductsNoSKU`
- `ProductsNoSKUItemNo`

Additionally, the default value for `ItemFieldForWooShort_Description` is now an empty string, and the default value for `SyncCustomerNumberAsMeta` is now `false`.

### Optimized WooCommerce Order Import for Multi-Item Inventory Updates

Fixed an issue where, after a WooCommerce order was imported into Counterpoint, some item quantities on the order would fail to update — inconsistently, and without a clear pattern.

- When an order is downloaded, Counterpoint performs a single multi-record update to adjust inventory for all affected items at once. However, the underlying inventory trigger (`USER_TR_WOOCOMMERCE_IM_INV_UI`) was only designed to process one item update at a time, causing some items to be missed when multiple items updated simultaneously.
- The trigger has been refactored to properly support multi-record updates, ensuring inventory status is updated consistently for every item — including both parent and child (variant) items — regardless of how many items are included in a single update.

### Corrected Error When Unflagging a Child Variant as an E-Commerce Item

Fixed an issue where unchecking the **WooCommerce Item** flag on a child variant in Counterpoint would cause the connector to throw an error instead of removing the item from WooCommerce.

- WooCommerce requires product variations to be deleted through a dedicated variations endpoint, rather than the standard product deletion endpoint used for regular items. The connector was previously using the standard endpoint for all deletions, which WooCommerce rejected for variations.
- The connector's item deletion logic has been updated so that child items are now correctly deleted as variations of their parent item, resolving the error and ensuring these items are properly removed from the website.
