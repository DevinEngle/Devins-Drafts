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

### Reduced Redundant Item Writes from the Inventory and Item Update Triggers

The Counterpoint inventory trigger (USER_TR_WOOCOMMERCE_IM_INV_UI) and item update trigger (USER_TR_WOOCOMMERCE_IM_ITEM_IU) have been refined so that they only write to the item record (IM_ITEM) when a relevant value actually changes. Previously, the inventory trigger could flag an item for sync every time its inventory record (IM_INV) was updated, even when no quantity changed, and both triggers could rewrite the sync flag on items that were already flagged. Because Counterpoint updates IM_INV often, this created unnecessary database activity.

The following changes were made:

- **Quantity changes only:** The inventory trigger now only flags an item for sync when a quantity column is part of the update. Updates to unrelated values, such as record lock stamps, replication flags, and maintenance dates, no longer flag the item.
- **Ecommerce location only:** The inventory trigger now only flags an item for sync when the updated inventory record is at the ecommerce location.
- **Sync flag:** USER_WOOCOMMERCE_STAT is now only set to 1 when it is not already 1, on both the item record (IM_ITEM) and the parent item table (USER_WOOCOMMERCE_PARENT_IM_ITEM). Items that are already flagged for sync are no longer rewritten.
- **Ecommerce item checkboxes:** The item update trigger now only processes unchecked ecommerce items when the ecommerce item checkbox (IS_ECOMM_ITEM) or the WooCommerce item checkbox (USER_WOOCOMMERCE_ITEM) is part of the update. Editing other fields on an item no longer runs this step.

Expected item behavior is otherwise unchanged. Changing an item's quantity at the ecommerce location still flags it for sync, and editing an item still flags it for sync.

### More Reliable Sync Flagging for Parent Products and Quantity Changes

The inventory and item update triggers now flag items for sync more consistently in two situations that could previously leave website quantities out of date.

- **Parent products in bulk edits:** When several items are moved to different parent products in a single update, the item update trigger (USER_TR_WOOCOMMERCE_IM_ITEM_IU) now flags both the previous parent products and the new parent products for sync.
- **Quantity changes from other triggers:** When another trigger in Counterpoint writes a quantity change to the inventory record (IM_INV), the inventory trigger (USER_TR_WOOCOMMERCE_IM_INV_UI) now flags the item for sync the same way as any other quantity change.

### Fulfilled Orders Update to Completed in WooCommerce

When the **Send Fulfillments configuration setting** is turned on, orders that are fulfilled in Counterpoint are now reliably updated to the "Completed" status in WooCommerce.

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
