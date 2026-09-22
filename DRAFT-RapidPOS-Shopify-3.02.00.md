# Shopify Connector v3.02.00 Release Notes

_Release Date: September 23, 2026_

---

## New Functionality

### Visibility into Shopify Items Included in Promotional Price Groups

This applies **only to clients using the Calculated Prices configuration option** for Shopify Product Price (`ITEM_PRC_METH`). A new view and Counterpoint form now make it possible to see, directly in the user interface, which items have a Shopify Item Record and are also included in a promotional price group with a calculated price being synced to Shopify.

- A new `USER_VI_USER_SHOPIFY_PROMO_WRK` view has been added, built on the `USER_SHOPIFY_PROMO_WRK` table, to expose the items affected by Shopify promotional pricing.
- A new Shopify Promo Prices form and menu item display the promotional price group code, item number, item description, item category and subcategory, item price 1, promotional price method (amount or percent), the value of the amount or percent, the calculated price being pushed to Shopify, the promotional begin and end dates and/or no date flags, as well as the Shopify Promo Status (this is the sync status of the item specific to the promotional price work table).
- The Shopify Promo Prices menu item is located in the new Shopify Other folder in the Shopify menu.
- If a price rule is enabled AND the end date has not yet passed, then there will be a record on this table. Therefore, there will be a record even for promotional prices that have not yet started because they will be synced in the future. 

### Refresh Shopify Items by Promotional Price Group

This applies **only to clients using the Calculated Prices configuration option** for Shopify Product Price (`ITEM_PRC_METH`). A new menu item allows the items in a specific promotional price group to be resynced to Shopify on demand, without requiring a full item resync.

- A new Shopify Promo Prices Refresh menu item runs a custom program that refreshes only the Shopify items in a specified promotional price group, using the group code as a filter. 
- The new `USER_SP_SHOPIFY_REFRESH_PROMO_PRICES` stored procedure wraps `USER_SP_SHOPIFY_UPDATE_PROMO_PRICES` and accepts the group code as a parameter, so all of the Shopify items in that specific group are refreshed (their Shopify Promo (Sync) Status is set to 1. These items will then resync to Shopify on the next run of the connector. 
- The Shopify Promo Prices Refresh menu item is located in the new Shopify Other folder in the Shopify menu.

---

## Bug Fixes and Performance Enhancements

### Shopify Menu Items Reorganized into Folders

As more menu items have been added to the Shopify menu over time, they had accumulated into a single large, unorganized group. The Shopify menu has been reorganized into folders to make it easier to find related items.

- A new Shopify Configuration Tools folder contains Shopify Configuration, Shopify Custom Field Mapping, Shopify Locations, and Shopify Customer Matching Priority.
- A new Shopify Other folder contains Shopify Promo Prices and Shopify Promo Prices Refresh.
- The most frequently used items, Shopify Items, Shopify Bulk Item Setup, Shopify Items Status View, Shopify Item Variants, Shopify Customers, Mark All Shopify Messages as Read, and Run Shopify Connector, remain at the top level of the Shopify menu for quick access.

### Windows Service Not Recovering After a SQL Server Restart or Reboot

Previously, the connector's Windows Service opened a single SQL connection at startup and shared that same connection for the entire lifetime of the process. If the connection was broken, for example by a SQL Server restart, reboot, or network interruption, the connector had no way to reconnect. Every scheduled sync for every configured account would continue to fail until a user manually restarted the Windows Service.

- The connector's `CommonService`, `CustomerInterface`, `ItemInterface`, `OrderInterface`, and `RunService` classes now use an `IDbConnectionFactory` to open a new database connection at the start of every sync cycle, and dispose of it when the cycle finishes, instead of sharing one connection for the life of the Windows Service process.
- If the SQL Server connection is lost, only the sync cycle in progress at that moment is affected. The next scheduled cycle automatically opens a new, healthy connection, so the Windows Service no longer needs to be manually restarted to recover.
- A related timing issue has also been corrected so that two overlapping scheduled runs can no longer both start at the same time while a previous run is still finishing.

### Incorrect Run Type Logged When a Sync Job Is Skipped

Previously, in `CustomerInterface.cs`, `ItemInterface.cs`, and `OrderInterface.cs`, when a scheduled sync job was skipped because a previous run of that same job was still in progress, the logged message showed the run type from the last successful run instead of the run type of the job that was actually skipped. This made the "already running" log messages misleading when reviewing connector logs.

- The already-running guard message in `CustomerInterface.cs`, `ItemInterface.cs`, and `OrderInterface.cs` now logs the run type of the call that was actually rejected, rather than the leftover run type from the previous successful run.

### SQL Install Script Now Removes Leftover Indexes Referencing the Deprecated USER_SHOPIFY_ID Column

As part of the ongoing removal of the deprecated `USER_SHOPIFY_ID` field, some client databases had custom indexes referencing that column on tables outside of `USER_SHOPIFY_ITEMS` and `USER_SHOPIFY_CUSTOMERS`. Because the install script did not previously account for these additional indexes, the script's column drop step could fail for clients with one of these leftover indexes in place.

- The install script now dynamically identifies and drops any remaining non-clustered, non-primary-key indexes that reference the deprecated `USER_SHOPIFY_ID` column, on tables other than `USER_SHOPIFY_ITEMS` and `USER_SHOPIFY_CUSTOMERS`, before the column itself is dropped.
- This prevents the install script's column drop step from failing when a client has a custom index referencing `USER_SHOPIFY_ID` that was not explicitly accounted for elsewhere in the script.

### Additional Indexes Added for Shopify Item and Customer Lookups

Queries filtering or sorting by item number or customer number on Shopify-related tables previously relied on composite indexes built for `ACCOUNT_NAME` plus `ITEM_NO` or `CUST_NO`. Dedicated single-column indexes have been added to improve performance for queries that do not also filter by account name.

- A new `IX_USER_SHOPIFY_ITEMS_ITEM_NO` index has been added to the `USER_SHOPIFY_ITEMS` table on the `ITEM_NO` column.
- A new `IX_USER_SHOPIFY_CUST_CUST_NO` index has been added to the `USER_SHOPIFY_CUST` table on the `CUST_NO` column, including the `USER_SHOPIFY_STAT` column, to optimize queries that filter or sort by customer number.
- Both indexes are created conditionally, so they will not be duplicated on databases that already have them.
