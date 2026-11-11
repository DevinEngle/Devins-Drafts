# Shopify Connector v3.02.04 Release Notes

_Release Date: November 11, 2026_

---

## New Functionality

### Updating Customer Email Addresses from Counterpoint to Shopify

A customer's **Email Address 1** can now be changed in Counterpoint even when that customer is already linked to a Shopify customer.

- This functionality only applies to clients who have the **Customers Up** Shopify configuration option enabled (`USER_SHOPIFY_CONFIG.CUSTOMERS_UP`).
- Previously, Shopify did not allow customer email addresses to be changed through the connector, so Counterpoint blocked the change with the message "Cannot update Email Address 1 because a Shopify Customer record is defined for this customer (######) and Shopify does not allow email address updates."
- Shopify now supports this update, so a customer's email address no longer needs to be corrected separately in each system.

How it works:

- When **Email Address 1** (`AR_CUST.EMAIL_ADRS_1`) is changed on a Counterpoint customer that has a Shopify Customer Record (`USER_SHOPIFY_CUST`), the customer is automatically flagged to sync.
- On the next connector run, the email address on the matching Shopify customer is updated.

Limitations:

- Shopify requires every customer to have a unique email address. If the newly updated email address is already used by another Counterpoint customer that has a Shopify Customer Record, the change is blocked (by the `USER_TR_SHOPIFY_AR_CUST_IU` trigger) so that it is not sent to Shopify only to be rejected. The user sees the message "Cannot update Email Address 1 for Customer (######) because [email address] is already used by Shopify-linked Customer (######)." The user can then review both customers and merge or edit them.

---

## Bug Fixes and Performance Enhancements

### Inactive Items Appearing Stuck in the Shopify Items Status View

Inactive Counterpoint items are not synced to Shopify by design. However, when an inactive item was flagged to sync, the connector moved it to a sync status of 2 (in progress) along with the other flagged items, and then skipped it during processing. As a result, these items stayed at a sync status of 2 and were counted in the **Shopify Items Status View**, which made it appear that they were stuck or failing to sync.

- The Shopify Items Status View (`USER_VI_SHOPIFY_ITEMS_BY_STAT`) now excludes inactive items (`IM_ITEM.STAT` = 'V'), so the counts for each sync status (`USER_SHOPIFY_STAT`) include only items that the connector actually processes.

### Reduced Redundant Item Writes from the Inventory and Item Update Triggers

The Counterpoint inventory trigger (`USER_TR_SHOPIFY_IM_INV_U`) and item update trigger (`USER_TR_SHOPIFY_IM_ITEM_U`) have been refined so that they only write the Shopify sync flag when it needs to change. Previously, the inventory trigger flagged an item for sync every time its inventory record (`IM_INV`) was updated, even when no quantity changed, and both triggers could rewrite the sync flag on items that were already flagged. Because Counterpoint updates `IM_INV` and `IM_ITEM` often, this created unnecessary database activity and unnecessary product updates to Shopify.

The following changes were made:

- **Quantity changes only:** The inventory trigger now only flags an item for sync when a quantity column (`QTY_*`) is part of the update, or when the include in ecommerce quantity column (`USER_INCL_IN_ECOMM_QTY`) is part of the update. That column controls which locations count toward the Shopify quantity. Updates to unrelated values, such as bin locations, minimum and maximum quantities, count information, and costs (`GL_VAL`, `LST_COST`), no longer flag the item.
- **Sync flag:** `USER_SHOPIFY_STAT` is now only set to 1 when it is not already 1, in both triggers. Items that are already flagged for sync are no longer rewritten. This also means that large item updates lock fewer Shopify Item Records, which reduces the chance of deadlocks.

Expected item behavior is otherwise unchanged. Changing an item's quantity still flags it for sync, including items that are not yet linked to a Shopify product, and editing an item still flags it for sync.

### Shopify Triggers Causing Item Updates to Fail on Offline Stations After a Rebuild

Two Shopify connector triggers, `USER_TR_IM_ITEM_SHOPIFY_ITEMS_COPY_FROM` and `USER_TR_USER_SHOPIFY_BULK_ITEM_SETUP_IU`, use SQL Server XML features that only work when the `QUOTED_IDENTIFIER` setting is ON. When an offline station (register) database is rebuilt, Counterpoint recreates all triggers with `QUOTED_IDENTIFIER` OFF. On those stations, this could cause item and price updates sent from the Counterpoint server to fail, so new or changed items did not appear on the register.

- These triggers now skip processing when they run on an offline station. The connector works only with the Counterpoint server database, so these triggers do not need to run on registers.
- The updated triggers reach offline stations automatically. No manual action is required on any register.

### Gridded Item Grid Cell Changes Not Syncing to Shopify

Previously, some changes to the grid cells of a gridded item in Counterpoint were not reaching Shopify. Grid cell additions and deletions in Counterpoint now sync to Shopify correctly.

Details: The connector builds a gridded item's Shopify variants from the Shopify Item Variants table (`USER_SHOPIFY_ITEM_VARIANTS`), which holds one record for each grid cell at each location mapped to Shopify. The triggers that keep this table in sync with the item's grid cells (`IM_INV_CELL`) have been corrected, and the connector now repairs any missing records before it syncs the item.

**New grid cells now sync to Shopify.**

- Previously, when a new dimension combination was stocked, the item was flagged to sync, but no variant record was created for the new grid cell, so the new variant was never created in Shopify.
- The connector now adds any missing variant records for the item's grid cells before syncing it.

**Deleted grid cells are now removed from Shopify.**

- Previously, the `USER_TR_SHOPIFY_IM_INV_CELL_D` trigger removed the variant records for a deleted grid cell but did not flag the item to sync, so the variant remained on Shopify until a later change, such as a sale, flagged the item.
- The trigger now flags the item, but only for the Shopify accounts where a variant record was actually removed. Deleting a grid cell at a location that is not mapped to Shopify does not flag the item.

**Items with three grid dimensions no longer lose other variants.**

- Previously, the triggers did not compare the third dimension (`DIM_3_UPR`) when matching variant records, so deleting one grid cell also deleted the variant records for every other grid cell that shared the same first and second dimension values. The connector then removed those variants from Shopify and later recreated them with new Shopify IDs.
- This has been corrected in the `USER_TR_USER_SHOPIFY_ITEMS_IU`, `USER_TR_USER_SHOPIFY_LOC_I`, and `USER_TR_SHOPIFY_IM_INV_CELL_D` triggers.
- Items with one or two grid dimensions were not affected.
- Variant records that were previously lost because of this issue are restored automatically the next time each affected item syncs. No action is required.

### Deadlocks When Syncing Items During Large Item Updates

Previously, if a large update to Counterpoint items (`IM_ITEM`) ran at the same time the connector was reading a Shopify Item Record, the two processes could block each other in SQL Server (a deadlock). SQL Server resolved the deadlock by stopping the connector's read, so the item did not sync during that run.

- The `userFnShopifyItemDescription` function, which supplies the Shopify product title (`USER_SHOPIFY_PRODUCT_TITLE`), and the connector's query that loads a single Shopify Item Record no longer wait on locks held by other updates to `IM_ITEM`. This matches how the connector's other Shopify functions and queries already read item data.
- Large item updates now also lock fewer Shopify Item Records. For details, see Reduced Redundant Item Writes from the Inventory and Item Update Triggers.

### Shopify Orders Not Fully Committed After a Deadlock

After the connector saves a downloaded Shopify order in Counterpoint, it runs the Counterpoint commit step (`USP_PS_TE_COMMIT_DOC`) to finalize the document. Previously, if this step was stopped by a deadlock with another process, such as a large update to Counterpoint items, the error was only logged. Because the order had already been saved, later connector runs treated it as already downloaded and skipped it, so the order remained in Counterpoint without being committed and was never retried.

- The commit step now runs at a high deadlock priority, so that SQL Server normally stops the other process instead of the order commit. The priority returns to normal as soon as the commit step finishes, so other connector activity is not affected.
- If the commit step is still stopped by a deadlock, the connector tries it up to three times in total, waiting one second and then two seconds between attempts. Any other error is handled the same way as before.
