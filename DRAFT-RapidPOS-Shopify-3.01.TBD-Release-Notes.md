# Shopify Connector vTBD Release Notes

_Release Date: September 8, 2026_

---

## Bug Fixes and Performance Enhancements

### Adjusted Shopify Station and EC_SHOPIFY Customer Configuration for Better Performance

The default configuration for the Shopify Station (usually 201-01) and for the `EC_SHOPIFY` customer record, which is used as a template whenever a new customer is created by the Shopify connector, has been adjusted for better performance.

- The install script now sets the Shopify Station's `Begin Tickets At` setting to Lines and unchecks `Use Default Customer`. This allows the Touchscreen application to open at this station without prompting for a customer selection, so a user can see orders for any customer instead of having the order list filtered to a single customer number.
- The `EC_SHOPIFY` customer will now always have `Allow Tickets` and `Allow Orders` checked by default, ensuring that customers created from the `EC_SHOPIFY` template have the ability to process tickets and orders.

### Automatic Detection of Deprecated Field References in Custom Database Objects

As database fields used by the connector are renamed or retired over time, custom triggers and stored procedures created outside the standard install script can continue to reference the old field names without anyone noticing.

- After each initial CI/CD deployment, the release pipeline now scans triggers and stored procedures that are outside the standard install script, meaning client-customized database objects, for references to deprecated Shopify connector fields.
- If a deprecated field reference is found in one of these custom objects, an email notification is sent in addition to the pipeline log message.

### Promotional Prices Not Refreshing When a Price Group Is Re-enabled

This applies only to clients using the Calculated Prices configuration option for Shopify Product Price (`ITEM_PRC_METH`). For those clients, if a Counterpoint price group used for Shopify promotional pricing was disabled and then re-enabled, the `USER_SHOPIFY_PROMO_WRK` table did not refresh to reflect the group's current promotional prices, so those prices did not sync to Shopify.

- The `USER_TR_SHOPIFY_IM_PRC_RUL_U` trigger has been corrected so that it properly calls the `USER_SP_SHOPIFY_UPDATE_PROMO_PRICES` stored procedure when promotional price rules change, including when a previously disabled price group is enabled again. This keeps the `USER_SHOPIFY_PROMO_WRK` table, and the promotional prices synced to Shopify from it, up to date.

### Friendly Error Message for Misconfigured Point of Sale Sales Channel

Previously, if a Shopify Item Record had Sales Channel - Point of Sale enabled (`USER_SHOPIFY_ITEMS.USER_SHOPIFY_SALES_CHANNEL_POS` = 'Y') for a store where the Point of Sale sales channel does not exist in Shopify, the connector failed with a generic technical error (a null reference exception) when syncing that item. The affected item would remain stuck without receiving updated price and quantity information.

- The item publication logic has been refactored to add checks for missing "Point of Sale" and "Online Store" sales channel publication data, which prevents this error from occurring.
- When the Point of Sale sales channel is not configured for a store, the connector now logs a friendly message identifying the affected item and automatically disables Sales Channel - Point of Sale for that item, so it can continue to sync normally.
