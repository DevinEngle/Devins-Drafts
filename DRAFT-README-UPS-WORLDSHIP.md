# Rapid POS UPS WorldShip ODBC Support
Updated 10/1/2026

---

Rapid provides support for the UPS WorldShip Desktop app to enable the transfer of data between Counterpoint and UPS Worldship, allowing for faster creation of shipping labels. The connector syncs information from **order release tickets** that include a ship-to address.

---

## Minimum System Requirements:
- Minimum Counterpoint version: **8.5.6.2**  
- Minimum SQL Server version: **2016**  
- Minimum Supported Operating System version: **Windows Server 2016** or **Windows 11 Pro** 
- Minimum PowerShell version: **5.1**  

If you would like the UPS WorldShip ODBC app but your system does not meet these minimum requirements, please consult your Care Team Lead (vCIO) for an upgrade quote.

---

## Table of Contents
- [Minimum System Requirements](#minimum-system-requirements)

---

## Section 1: Overview of the information provided to UPS WorldShip by the ODBC Connection

The custom view makes information from one type of Counterpoint document accessible to UPS WorldShip: order release tickets with a ship-to address.

With ODBC, WorldShip connects to the database directly, over the store's local network, and reads a custom table that Rapid makes available to it that contains information from release tickets. This program relies on Windows 32-bit ODBC tool so nothing needs to be installed other than a driver (so that the UPS WorldShip PC can communicate with Microsoft SQL Server) and the UPS WorldShip desktop app which the client will download.

### Order Release Tickets

- When an order (or part of an order) is released in Counterpoint, a release ticket is created.
- Only order release tickets that contain a ship-to address are provided to UPS WorldShip.
- Orders and regular tickets are not sent to UPS WorldShip.

### Data Flow to UPS WorldShip

The ODBC connection is a Windows program that continuously makes information in the Counterpoint database available to be queried by UPS WorldShip. As soon a release ticket is created in Counterpoint, it is available to UPS WorldShip.

## Section 2: Mapping of specific fields sent to UPS WorldShip for release tickets

### Release Ticket Header Fields

| Counterpoint Field | UPS WorldShip Section | UPS WorldShip Field |
|---|---|---|
| Document ID | Shipment Information | Unique Shipment Identifier |
| Ticket Number | Shipment Information | Reference 1 |
| Ship-To Email Address 1 | Shipment Information | Recipient Email Address |
| Ship-To Customer Number | Ship To | Customer ID |
| Ship-To Name | Ship To | Company or Name |
| Ship-To Address 1 | Ship To | Address 1 |
| Ship-To Address 2 | Ship To | Address 2 |
| Ship-To Address 3 | Ship To | Address 3 |
| Ship-To Country | Ship To | Country/Territory |
| Ship-To Zip Code | Ship To | Postcode |
| Ship-To City | Ship To | City or Town |
| Ship-To State/Province | Ship To | State/Province/County |
| Ship-To Phone 1 | Ship To | Telephone |
| Ship-To Email Address 1 | Ship To | Email Address |

No release ticket line data is provided to UPS WorldShip.

## Section 3: Special Note on State/Province Abbreviations

UPS WorldShip requires all state and province abbreviations to be formatted as valid two-letter codes.

You can review the current state and province code list in UPS WorldShip's documentation:
https://www.ups.com/worldshiphelp/WSA/ENU/AppHelp/mergedProjects/CORE/Codes/State_Province_Codes.htm

Be careful to use two-letter abbreviations such as KS and MO. Do not use full states such as Kansas or Missouri, as these are not valid in UPS WorldShip.

## Section 4: Special Note on Country Codes

UPS WorldShip requires all country codes to be formatted as valid two-letter ISO country codes.

You can review the current ISO country code list in UPS WorldShip's documentation:
https://www.ups.com/worldshiphelp/WSA/ENU/AppHelp/mergedProjects/CORE/Codes/Country_Territory_and_Currency_Codes.htm

Be careful to use two-letter abbreviations such as US and CA. Do not use three-letter codes such as USA or CAN, as these are not valid in UPS WorldShip.

## Conclusion

The UPS WorldShip ODBC connection streamlines the exchange of order shipping information (from release tickets) to UPS Worldship. This reduces manual entry and ensures shipments move efficiently.

If you have questions about setup, troubleshooting, or advanced options, please reach out to support.
