# BigCommerce v2.02.01 Release Notes
_Release Date: August 3, 2026_

---

## New Functionality

### Mark Alerts as Read in Message Center After Days
Added a new configuration option that automatically marks connector alerts as read after a defined number of days, reducing repeated Message Center pop-up notifications.
- The default value is 3 days, meaning any `EC_BIGCOMM` messages will automatically be marked as read in Counterpoint after 3 days.
- Messages marked as read in this way will still be visible in the Message Center — they simply will no longer trigger a pop-up notification.

### Mark All Messages as Read Menu Button
Added a **Mark All Messages as Read** menu button under the BigCommerce menu, giving users a quick way to clear all pending alert pop-ups regardless of message age.
- Pressing this button marks all messages as read immediately, stopping them from popping up in Counterpoint.
- This is especially useful when multiple alerts are written to the Message Center at once — for example, if a user logs into Counterpoint and receives a high volume of pop-ups that impede their workflow, they can quickly hit this button to stop the interruptions.
- After clearing the pop-ups, the user can still visit the Message Center to review the messages and determine next steps.

---

## Bug Fixes and Performance Enhancements

### Faster Account Creation During Connector Install
In an earlier release, the connector was given the ability to configure GL account numbers during installation, but not create them. Now, the connector's install script has been enhanced so that it can automatically create the necessary accounting numbers during initial installation of the connector too.
- Accounts are only created when a client is using Rapid POS's default account setup.

| Function | Default Account | Purpose |
|---|---|---|
| BigCommerce Tender | 1304 | Records BigCommerce customer payments |
| BigCommerce Deposit Liabilities | 2254 | Records deposits collected on BigCommerce orders |
| BigCommerce Sales Tax Payable | 2314 | Records BigCommerce tax amounts |
| Shipping | 4950 | Records shipping charges |
| BigCommerce Needs Investigation | 9904 | Catch all account for unexpected activity |

- If the client uses profit centers, the script creates the required Main accounts (if not already present), then creates Posting accounts only for the accounts that were newly created.
- If the client does not use profit centers, the script creates the required accounts directly as both Main and Posting accounts, if not already present.
- For more details, review: [BigCommerce Connector GL Account Numbers](https://github.com/Rapid-POS/Rapid-Counterpoint-BigCommerce-Connector/blob/main/BigCommerce-Connector-GL-Account-Numbers.md)

### Corrected Inventory Sync Status Update on Nested Triggers
Fixed an issue where certain inventory updates were occasionally not flagged for sync with BigCommerce, so product quantity could end up being incorrect on the website.
- When an intake transaction updates inventory, the `IM_INV` (inventory) record is updated twice in quick succession. This occasionally caused the `USER_TR_BIGCOMMERCE_IM_INV_U` trigger to detect a nested call (`TRIGGER_NESTLEVEL()` greater than 1) and exit before it could set `IM_ITEM.USER_BIGCOMMERCE_STAT` to 1, the flag that tells the connector an item needs to sync.
- The nest-level check has been removed so the trigger now sets the sync flag even when triggered by a nested inventory update.
