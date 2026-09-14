# Shopify Connector vTBD Release Notes

_Release Date: September 16, 2026_

---

## Bug Fixes and Performance Enhancements

### Products Intermittently Failing to Add to Cart Due to Redundant Sales Channel Republishing

Previously, on every sync, the connector re-sent a Shopify `productPublish` mutation for the Point of Sale and Online Store sales channels with a new `publishDate`, even when a channel was already published and its state had not changed. Because Shopify's Online Store channel triggers downstream work such as storefront cache invalidation and search reindexing, these repeated, unnecessary publish calls could cause a temporary mismatch between the Shopify Admin interface, the Shopify API, and the live storefront. During this window, a product could appear correctly configured in the Shopify Admin interface but be unavailable on the storefront, which prevented customers from adding it to their cart.

- The connector's publication logic in `ItemInterface.cs` has been updated so that a sales channel is only sent to Shopify's `productPublish` mutation when its published state has actually changed, rather than being resent on every sync regardless of whether anything changed.
- This change applies to both the Point of Sale and Online Store sales channels.
- This resolves an issue in which products would intermittently become unavailable for purchase on a client's Shopify storefront, then recover on their own a short time later, with no corresponding change made in Counterpoint.

### Promotional Prices Not Syncing for Shopify Items Created After a Price Rule Is Enabled

This applies only to clients using the Calculated Prices configuration option for Shopify Product Price (`ITEM_PRC_METH`). For those clients, if a Counterpoint price rule used for Shopify promotional pricing was already enabled, and a Shopify Item Record was created afterward for an item covered by that rule, the item's promotional price did not sync to Shopify. This happened because the promotional pricing data was only refreshed when the price rule's enabled flag was toggled, its begin or end dates were changed, or the rule itself was edited, not when a new Shopify Item Record was added for an item under an already enabled rule.

- A new `USER_TR_USER_SHOPIFY_ITEMS_PROMO_PRICE_I` after insert trigger has been added on `USER_SHOPIFY_ITEMS` to update the related `IM_PRC_RUL_BRK` record whenever a Shopify Item Record is created after its price rule has already been enabled, so the item's promotional price syncs to Shopify correctly.
- Redundant `TRIGGER_NESTLEVEL()` checks have also been removed from the `USER_TR_SHOPIFY_IM_PRC_RUL_BRK`, `USER_TR_SHOPIFY_IM_PRC_GRP_U`, and `USER_TR_SHOPIFY_IM_PRC_GRP_D` triggers to streamline the underlying logic.
