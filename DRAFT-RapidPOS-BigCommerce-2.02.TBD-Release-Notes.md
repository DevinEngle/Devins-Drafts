# BigCommerce Connector v2.02.TBD Release Notes

_Release Date: October 27, 2026_

---

### Dedicated SQL Login for the Connector

The connector now connects to the Counterpoint database with its own dedicated SQL Login, instead of sharing the main SQL Login used by Counterpoint. Because the connector's database activity can now be identified separately from Counterpoint's, Rapid can limit how much of the SQL Server's CPU the connector uses, so that connector activity does not slow down Counterpoint for store users.

- The SQL Login is created automatically during the connector upgrade. 
- The new login receives the same server and database permissions as the existing Counterpoint SQL Login, so the connector continues to work the same way as before.
- A unique, strong password is generated for the login and is stored only in encrypted form in the connector's configuration. A new password is generated each time the connector is upgraded.
- Do not change the password for this SQL Login or remove the login. The connector depends on it, and its password is managed automatically by the upgrade process.

---

## Bug Fixes and Performance Enhancements

### Reduced Redundant Item Writes from the Item Update Trigger

The Counterpoint item update trigger (USER_TR_BIGCOMMERCE_IM_ITEM_U) has been refined so that it only writes to the item record (IM_ITEM) and the BigCommerce deletion queue (USER_BIGCOMMERCE_ITEM_DEL) when a relevant value actually changes. Previously, the trigger could rewrite BigCommerce fields every time an item was saved, even when those values were already correct. This created unnecessary database activity, which was most noticeable during mass updates.

The following changes were made:

- **Sync flag:** USER_BIGCOMMERCE_STAT is now only set to 1 when it is not already 1. Items that are already flagged for sync are no longer rewritten.
- **Visibility:** USER_BIGCOMMERCE_VISIBLE is now only updated when the value changes. For example, changing an item from one inactive status to another (such as D to V) no longer causes an extra write, because the item remains hidden (N).
- **Adding items to the deletion queue:** Items are now only added to USER_BIGCOMMERCE_ITEM_DEL when the ecommerce item checkbox (IS_ECOMM_ITEM) is part of the update. Items that were never synced to BigCommerce (USER_BIGCOMMERCE_ID is empty) are no longer added to the deletion queue when the checkbox is unchecked.
- **Removing items from the deletion queue:** Items are now only removed from USER_BIGCOMMERCE_ITEM_DEL when the ecommerce item checkbox changes from N to Y. Editing other fields on an item no longer removes an existing deletion queue entry.

Expected item behavior is otherwise unchanged. Editing an item still flags it for sync, changing an item between active and inactive statuses still updates its visibility in BigCommerce, and unchecking the ecommerce item checkbox on a synced item still queues it for removal from BigCommerce. 
