# Rapid POS Shopify Connector {{Version}} Release Notes

_Release Date: {{Month Day, Year}}_

---

## New Functionality

### Item Matching – Counterpoint Item Number to Shopify SKU for Items Up

Items Up now matches existing Shopify products using the **Counterpoint Item Number** and **Shopify SKU**, instead of relying solely on the Shopify ID.

- If no match is found by Shopify ID, the connector now falls back to matching by SKU before treating the item as new.
- This makes it easier to connect existing Shopify products to Counterpoint — set the Shopify SKU to match the Item Number, create the Shopify Item Record, and the connector finds the match automatically. No more looking up and entering the Shopify ID by hand.
- Important: when matching to an existing Shopify product — especially on the first sync, but on any sync going forward — make sure the Shopify Item Record is fully filled in. In some cases, blank fields in Counterpoint (e.g., shipping weight) will overwrite existing values in Shopify. (The fields that will be overwritten vary based on individual client configuration settings.)

### Configurable Default Account Name for Shopify Item Records

Clients can now set a default **Account Name** for new Shopify Item Records instead of entering it manually every time. For example, Rapid Garden Center can default to `RAPID`.

- This default is included in the install script for this upgrade and will be applied automatically. Clients with more than one account will still need to enter the value manually.
- Applies to manual Shopify Item Record creation; does not affect Bulk Item Creation, where Account Name is still entered per batch.
- Set via a data dictionary default on `USER_SHOPIFY_ITEMS.ACCOUNT_NAME`, configurable per client.

---

## Bug Fixes and Performance Enhancements

### Customers Up Error – Object Reference Not Set to an Instance of an Object

{{Awaiting details from Ticket #706359 — description, root cause, and fix to be added here.}}

### Double Quotes in Item Description Causing Sync Errors

Clients who include the item description in their Shopify handle formula (the final segment of a product's URL) could hit a sync error if that description contained a double quote (`"`) — the connector couldn't generate a valid handle. This has been fixed.

- Only affected newly created Shopify Item Records where the description used to build the handle included a double quote.
- The handle-generation logic now correctly parses double quotes so these items sync successfully.

### Refund Processing Loop When Customer Not Found

Previously, if a customer record couldn't be found in Counterpoint during a Shopify refund, the sync would fail and the refund would loop instead of completing.

- The connector now falls back to a default "walk-in" customer (using the Walk-In Customer Number in Store Config) when no match is found.
- Refunds now complete successfully instead of getting stuck in a retry loop.
