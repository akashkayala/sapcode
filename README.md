"-----------------------------------------------------------------------
"* Fill Vendor Central Data for Vendor Master Creation
"-----------------------------------------------------------------------
"1. Populate the structure VMDS_EI_CENTRAL_DATA with vendor master details.
"2. Move relevant LFA1 fields into the central vendor data section (general data).
"3. Set the change indicator for vendor header data (CENTRAL_DATA-X) to 'X'.
"4. Retrieve address details using the ADRC table based on LFA1-ADRNR (address number).
"5. Update the address change indicator as 'M' (Modify) for address data.
"6. Fill address fields such as street, city, postal code, and country from ADRC.
"7. Fetch and populate telephone details from ADR2 (telephone numbers).
"8. Retrieve fax number information from ADR3 and assign it to fax data fields.
"9. Extract teletex information from ADR4 and update corresponding structure fields.
"10. Retrieve telex number details from ADR5 and fill into communication data.
"11. Populate email details by reading ADR6 entries linked to the vendor address.
"12. Fetch remote mail information from ADR7 and update remote mail section.
"13. Retrieve printer details (output device) from ADR10 and assign appropriately.
"14. Populate FTP and URI details by reading ADR12 records associated with the vendor.
"15. Fill VAT registration and tax grouping fields from LFA1, and bank details from LFBK and TIBAN.
"-----------------------------------------------------------------------
