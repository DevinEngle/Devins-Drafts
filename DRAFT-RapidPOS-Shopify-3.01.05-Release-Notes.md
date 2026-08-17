# Shopify Connector v3.01.05 Release Notes

_Release Date: August 19, 2026_

---

## Bug Fixes and Performance Enhancements

### Error When Adding New Shopify Items

In our last release, a trigger nesting restriction was removed to fix an issue where gridded (variant) items weren't populating the `USER_SHOPIFY_ITEM_VARIANTS` table. That change introduced a new problem where adding a new Shopify item record could exceed the maximum number allowed for stored procedures, functions, triggers, or view nesting.

- The `TRIGGER_NESTLEVEL` check in the `USER_TR_USER_SHOPIFY_ITEMS_IU` trigger has been restored, which prevents the nesting-limit error when saving new Shopify items.
- The `USER_SHOPIFY_ITEM_VARIANTS` fix from August 12, 2026 is unaffected by this change. Variant records (barcode, SKU, and quantity) continue to be kept in sync through the dedicated `USER_TR_SHOPIFY_IM_INV_CELL_IU` and `USER_TR_SHOPIFY_IM_INV_CELL_D` triggers, which do not rely on the nested trigger execution that was reverted.
