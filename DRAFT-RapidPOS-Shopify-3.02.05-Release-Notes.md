# Shopify Connector v3.02.05 Release Notes

_Release Date: November 17, 2026_

---

## Bug Fixes and Performance Enhancements

### Custom Field Mapping Sending 0 for User-Defined Item Fields

Previously, when a user-defined item field in Counterpoint was mapped to a Shopify product metafield, the connector sent a value of 0 to Shopify instead of the field's actual value. Mapped user-defined item fields now send their correct values to Shopify.

- This only affected custom item columns added to `IM_ITEM`, such as `USER_USDA_ZONE`, that are mapped in the Shopify Custom Field Mapping table (`USER_SHOPIFY_CUSTOM_FIELD_MAPPING`).

### Install Script Failing When Creating Posting Accounts

During installation, the connector's SQL install script creates any posting accounts that the connector needs and that do not yet exist in Counterpoint (`SY_ACCT`). Previously, the script could stop with a duplicate account error when it created posting accounts for each profit center, which prevented the installation from completing.

- The script now checks for existing accounts before creating new ones, so it no longer attempts to create an account that already exists.
- Accounts entered in the install script settings are now validated before the script creates any posting accounts. The script can only create posting accounts from a main account that uses the profit center placeholder format.
- If an entered account does not exist and cannot be created automatically, the script displays a message such as "Account (4050-800) does not exist and is not a main account ending in the identified profit center placeholder. Create the account in Counterpoint first." The installer can then create the account in Counterpoint and run the script again.
- Existing installations are not affected. This issue only occurred while the install script was running.
