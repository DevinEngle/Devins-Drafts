# Shopify Connector v3.01.09 Release Notes

_Release Date: September 9, 2026_

---

## Documentation Updates

### Subcategory Filtering for Promotional Price Rules

The Using Calculated Prices guide did not previously document that a Promotional Price rule's item filter can target a **single item category paired with one of its _subcategories_**, in addition to filtering by item number or by item category alone. This capability has always been supported by the Calculated Price method, but was not previously documented.

If Calculated Prices is used for Shopify promotional pricing, and a promotion needs to target a specific subcategory within a category, see the [Using Calculated Prices guide](https://github.com/Rapid-POS/Rapid-Counterpoint-Shopify-Connector/blob/main/Using-Calculated-Prices.md) for configuration details.

---

## Bug Fixes and Performance Enhancements

### Adjusted Shopify Station and EC_SHOPIFY Customer Configuration for Better Performance

The default configuration for the Shopify Station (usually 201-01) and for the `EC_SHOPIFY` customer record, which is used as a template whenever a new customer is created by the Shopify connector, has been adjusted for better performance.

- The install script now sets the Shopify Station's `Begin Tickets At` setting to Lines and unchecks `Use Default Customer`. This allows the Touchscreen application to open at this station without prompting for a customer selection, so a user can see orders for any customer instead of having the order list filtered to a single customer number.
- The `EC_SHOPIFY` customer will have `Allow Tickets` and `Allow Orders` checked by default, ensuring that customers created from the `EC_SHOPIFY` template have the ability to process tickets and orders.

### Automatic Detection of Deprecated Field References in Custom Database Objects

As database fields used by the connector are renamed or retired over time, custom triggers and stored procedures created outside the standard install script can continue to reference the old field names without anyone noticing.

- During each CI/CD upgrade, the release pipeline now scans triggers and stored procedures that are outside the standard install script, meaning client-customized database objects, for references to deprecated Shopify connector fields.
- If a deprecated field reference is found in one of these custom objects, an email notification is sent in addition to the pipeline log message.

### Promotional Prices Not Refreshing When a Price Group Is Re-enabled

This applies only to clients using the Calculated Prices configuration option for Shopify Product Price (`ITEM_PRC_METH`). For those clients, if a Counterpoint price group used for Shopify promotional pricing was disabled and then re-enabled, the `USER_SHOPIFY_PROMO_WRK` table did not refresh to reflect the group's current promotional prices, so those prices did not sync to Shopify.

- The `USER_TR_SHOPIFY_IM_PRC_RUL_U` trigger has been corrected so that it properly calls the `USER_SP_SHOPIFY_UPDATE_PROMO_PRICES` stored procedure when promotional price rules change, including when a previously disabled price group is enabled again. This keeps the `USER_SHOPIFY_PROMO_WRK` table, and the promotional prices synced to Shopify from it, up to date.

### Promotional Prices Not Reverting When a Price Rule's Categories Are Edited

This applies only to clients using the Calculated Prices configuration option for Shopify Product Price (`ITEM_PRC_METH`). For those clients, if a Counterpoint price rule used for Shopify promotional pricing was edited after it had synced to Shopify, specifically by removing one of its categories, or by adding a subcategory qualifier to an existing category, the connector did not correctly remove the promotional price from items that were no longer covered by the rule. Those items kept showing the old promotional price on Shopify even though the rule no longer applied to them.

- The `USER_TR_SHOPIFY_PROMO_WRK_D` and `USER_TR_SHOPIFY_IM_PRC_RUL_U` triggers have been corrected, and a new `USER_TR_SHOPIFY_IM_PRC_GRP_D` trigger has been added, so that items removed from a price rule's category or subcategory scope have their promotional price cleared correctly.

### Friendly Error Message for Misconfigured Point of Sale Sales Channel

Previously, if a Shopify Item Record had Sales Channel - Point of Sale enabled (`USER_SHOPIFY_ITEMS.USER_SHOPIFY_SALES_CHANNEL_POS` = 'Y') for a store where the Point of Sale sales channel does not exist in Shopify, the connector failed with a generic technical error (a null reference exception) when syncing that item. The affected item would remain stuck without receiving updated price and quantity information.

- The item publication logic has been refactored to add checks for missing "Point of Sale" and "Online Store" sales channel publication data, which prevents this error from occurring.
- When the Point of Sale sales channel is not configured for a store, the connector now logs a friendly message identifying the affected item and automatically disables Sales Channel - Point of Sale for that item, so it can continue to sync normally.

### Order Download Failure When a Shopify Line Item Incorrectly Matches a Gridded Item

Previously, when downloading orders from Shopify, if an order line's Shopify SKU matched a barcode linked to a gridded item in Counterpoint, but the connector could not match the Shopify variant to one of that item's grid cells, it failed with a generic technical error (a null reference exception). This caused the connector's order download process to stop entirely, so any other orders still waiting to be downloaded in that batch were not downloaded either.
 
This can happen when the Match by Barcode setting (`MATCH_BY_BARCOD`) is enabled and is used as a fallback after the Shopify Variant ID and Shopify Product ID fail to find a match. If a Shopify SKU falls back to matching a barcode that belongs to a gridded (variant) item in Counterpoint, the connector also needs the Shopify variant ID to match one of the variant IDs stored against that item's grid cells, so it can determine which grid cell the order line belongs to. When no such variant ID match exists, the connector cannot reliably assign the grid cell.

- The connector now checks for this condition and, when it is detected, skips only the affected order instead of stopping the connector's order download process entirely.
- A friendly message is now logged identifying the Shopify SKU and product along with the mismatched Counterpoint item, so the cause is clear and the remaining orders in the batch continue to download normally.
