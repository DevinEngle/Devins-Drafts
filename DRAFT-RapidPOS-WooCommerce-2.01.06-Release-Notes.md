# WooCommerce Connector v2.01.06 Release Notes

_Release Date: November 10, 2026_

---

## New Functionality

### Dedicated SQL Login for the Connector

The connector now connects to the Counterpoint database with its own dedicated SQL Login, instead of sharing the main SQL Login used by Counterpoint. Because the connector's database activity can now be identified separately from Counterpoint's, Rapid can limit how much of the SQL Server's CPU the connector uses, so that connector activity does not slow down Counterpoint for store users.

- The SQL Login is created automatically during the connector upgrade.
- The new login receives the same server and database permissions as the existing Counterpoint SQL Login, so the connector continues to work the same way as before.
- A unique, strong password is generated for the login and is stored only in encrypted form in the connector's configuration. A new password is generated each time the connector is upgraded.
- Do not change the password for this SQL Login or remove the login. The connector depends on it, and its password is managed automatically by the upgrade process.

---

## Bug Fixes and Performance Enhancements

### Reduced Database Load from Inventory Changes

The connector uses database triggers in Counterpoint to flag items that need a quantity update on WooCommerce. (A trigger is a small piece of SQL code that runs automatically whenever a table is changed.) These triggers now skip work that does not affect website stock, which reduces the load on the SQL Server and helps keep Counterpoint responsive for store users. Items are still flagged for sync every time their quantity changes.

- Counterpoint updates the inventory table (IM_INV) often for reasons that have nothing to do with stock, such as record lock stamps, replication flags, and maintenance dates. The inventory trigger now flags an item only when a quantity column is updated, and only when the change is at the ecommerce location.
- Items that are already flagged for sync are no longer written to the item table (IM_ITEM) again. Each write to IM_ITEM also runs the other triggers on that table, so skipping these repeat writes prevents a single small inventory change from causing a large amount of database activity.

### More Reliable Sync Flagging for Parent Products and Quantity Changes

The connector now flags items for sync more consistently in two situations that could previously leave website quantities out of date.

- When several items are moved to different parent products in a single bulk edit, both the previous parent products and the new parent products are now flagged for sync.
- When another process in Counterpoint (for example, another database trigger) writes a quantity change to the inventory table, the item is now flagged for sync the same way as any other quantity change.

### Fulfilled Orders Now Update to Completed in WooCommerce

When the Send Fulfillments setting is turned on, orders that are fulfilled in Counterpoint are now reliably updated to the "Completed" status in WooCommerce.

- Each order is marked as sent in Counterpoint only after WooCommerce confirms that it accepted the update. If the update does not go through, the order is sent again on the next connector run.
- Each fulfilled order is tracked individually, so the status of one order does not affect any other order.
- Fulfillments are sent to WooCommerce in batches of 100 orders, and any failures are recorded in the connector log as errors.
- REMINDER: Fulfillments are sent after the order is closed and posted in Counterpoint. An order that is not yet posted is sent on the first connector run after it is posted.

### Products Already Deleted in WooCommerce No Longer Stop the Sync

When the Delete Products setting is turned on, the connector moves a product to the WooCommerce trash after the item is no longer flagged as an ecommerce item in Counterpoint. If the product was already moved to the trash or deleted directly in WooCommerce, the connector now treats it as removed and continues with the rest of the sync, including inventory updates.

- The connector marks these products as processed in Counterpoint and records an informational message in the connector log.
- This also applies to products and variations that no longer exist in WooCommerce.
- REMINDER: The recommended way to remove a product from WooCommerce is to unflag the item as an ecommerce item in Counterpoint and allow the connector to move the product to the trash on its next full run.

### Improved Hiding of Products and Variations That Are No Longer Ecommerce Items

When an item is no longer flagged as an ecommerce item in Counterpoint, the connector hides the matching product or variation on WooCommerce. Hiding now works for all product types and for variations.

- For products, the connector now updates only the product's catalog visibility, instead of sending the entire product back to WooCommerce. This allows products of every type to be hidden successfully.
- Variations do not have a catalog visibility setting in WooCommerce, so the connector now hides a variation by setting its status to Private. If the item is flagged as an ecommerce item again, the variation is set back to Published during the next sync.
- If a product cannot be hidden, the connector log now includes the item number to make troubleshooting easier.

### Reliable Loyalty Point Adjustments

The connector processes loyalty point adjustments for several customers at the same time. Adjustments for different customers no longer conflict with each other or with Counterpoint, so each customer's adjustment is applied on the same connector run.

- Each loyalty point adjustment requires a new event number from Counterpoint. Adjustments now wait their turn for an event number, instead of occasionally receiving the same number as another adjustment and being skipped until a later run.
- If the SQL Server briefly blocks an adjustment because another process is using the same records, the connector now retries the adjustment automatically. Adjustments that were already applied are never applied twice.
