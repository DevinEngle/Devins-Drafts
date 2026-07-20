# RCTA vX.XX.XX Release Notes

_Release Date: TBD_

---

## Bug Fixes and Performance Enhancements

### Corrected Misleading "Invalid CC Address" Error When Saving a New Scheduled Report
Resolved an issue where saving a new scheduled report could fail with an error indicating an invalid CC address, even when the CC address field was empty.

### Improved Email Address Delimiter Handling for To and CC Fields
Expanded delimiter options for both the To Email Address and CC Email Address List fields.

- The **To email address** field now accepts a comma, a semicolon, or a combination of both as delimiters between addresses.
- The **CC email address list** field now accepts a comma, a semicolon, a line break, or any combination of these as delimiters between addresses.
  - Line breaks are supported only in the CC email address list field; the To email address field does not support line breaks.
