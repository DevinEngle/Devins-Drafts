# Rapid POS WooCommerce Connector v2.01.00 Release Notes

_Release Date: July 21, 2026_

---

## New Functionality

### FFL Cockpit Plugin Enhancement

FFL Cockpit is a WooCommerce plugin used by firearm retailers to manage compliance-related order fulfillment and vendor drop-shipping. Clients selling firearms online through the **FFL Cockpit** plugin for WooCommerce can now have the connector automatically identify drop-ship order lines fulfilled through that plugin, rather than requiring every order line to be manually swapped to a specific item in Counterpoint.

- A new configuration setting controls this behavior. It defaults to disabled, meaning existing clients will see no change in behavior unless this setting is explicitly enabled.
- Enabling this feature requires extra effort. Reach out to Rapid for more details and a quote.

### Null Value Support for Custom Field Mapping Table (Metafield, Product Property, or Attribute)

The connector will now sync a null value to WooCommerce (for custom-mapped fields) when the corresponding field in Counterpoint is cleared — a capability that was not previously available for fields managed through the **WooCommerce Custom Field Mapping** table.

- Previously, the connector skipped sending null values to WooCommerce for custom-mapped fields. This meant that once a value was set for a mapped Metafield, Product Property, or Attribute, it could not be unset from Counterpoint.
- The connector now sends null values specifically for fields tied to the custom field mapping table, allowing these values to be explicitly cleared in WooCommerce when the source field in Counterpoint is cleared.

---

## Bug Fixes and Performance Enhancements

### Optimized WooCommerce Order Import for Multi-Item Inventory Updates

Improved how the connector updates inventory during WooCommerce order import, ensuring item quantities update reliably across all items on an order — including both parent and child (variant) items.

- The inventory trigger (`USER_TR_WOOCOMMERCE_IM_INV_UI`) has been adjusted to support multi-record updates, ensuring inventory status is updated consistently for every item.

### Corrected Error When Unflagging a Child Variant as an Ecommerce Item (for clients using parent-child functionality)

Fixed an issue where unchecking the ecommerce/WooCommerce flag on a child variant in Counterpoint would cause the connector to throw an error instead of removing the item from WooCommerce.

- WooCommerce requires product variations to be deleted through a dedicated variations endpoint, rather than the standard product deletion endpoint used for regular items.
- The connector's item deletion logic has been updated so that child items are now correctly deleted as variations of their parent item, resolving the error and ensuring these items are properly removed from the website.

### Updated Fields Related to Parent-Child Functionality to Be Keyword Searchable (for clients using parent-child functionality)

The **WooCommerce Parent Item Lookup** screen now supports keyword searching, making it faster to find a parent item without needing to know its exact item number.

- Users can now enter a keyword in the **Keyword** field to search by **Parent Item #**, **Product Name**, or **Description** — a minimum of 3 characters is required to return results.
- Added keyword fields to the `USER_WOOCOMMERCE_PARENT_IM_ITEM` and `USER_WOOCOMMERCE_PRODUCT_OPTION_VALUE` tables to support this search functionality.

### Faster Account Creation During Connector Install

Enhanced the connector's install script so that it can automatically create the necessary accounting numbers during initial installation of the connector.

- Accounts are only created when a client is using Rapid POS's default account setup.
>| Function | Default Account | Purpose |
>|---|---|---|
>| WooCommerce Tender | 1302 | Records WooCommerce customer payments |
>| WooCommerce Deposit Liabilities | 2252 | Records deposits collected on WooCommerce orders |
>| WooCommerce Sales Tax Payable | 2312 | Records WooCommerce tax amounts |
>| Shipping | 4950 | Records shipping charges |
>| WooCommerce Needs Investigation | 9902 | Catch all account for unexpected activity |
- If the client uses profit centers, the script creates the required Main accounts (if not already present), then creates Posting accounts only for the accounts that were newly created.
- If the client does not use profit centers, the script creates the required accounts directly as both Main and Posting accounts, if not already present.
- For more details, review: https://github.com/Rapid-POS/Rapid-Counterpoint-WooCommerce-Connector/blob/main/WooCommerce-Connector-GL-Account-Numbers.md

### Deprecated Unused Configuration Fields

Removed five configuration fields from the connector's install process that were found to be unused by any current connector logic.

- `SyncCustomers`
- `OverrideExistingSalePrice`
- `UseEmailAsShipToEmail`
- `ImportProductsNoSKU`
- `ProductsNoSKUItemNo`

Additionally, the default value for `ItemFieldForWooShort_Description` is now an empty string, and the default value for `SyncCustomerNumberAsMeta` is now `false`.
