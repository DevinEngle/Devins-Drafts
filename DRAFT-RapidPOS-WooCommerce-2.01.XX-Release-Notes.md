# WooCommerce Connector v2.01.XX Release Notes

_Release Date: August 11, 2026_

---

## Bug Fixes and Performance Enhancements

### Friendlier Error Message for Product Variations Removed Directly in WooCommerce

If a product variation was deleted directly in WooCommerce before the sync could remove it, the connector now logs a plain-language (friendly) message instead of a technical error.

- The connector checks whether a variation flagged for deletion (via the `USER_WOOCOMMERCE_IM_ITEM_DEL` table in Counterpoint) still exists in WooCommerce before attempting to delete it.
- If the variation is already gone, the connector now logs a message such as: **Product variant [`WooCommerce SKU #`] (WooCommerce ID [`WooCommerce ID #`]) was already removed from WooCommerce.**
- This message appears once per affected item and does not interrupt the rest of the sync process.
