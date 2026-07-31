REPORT zvbap_to_xlsx_al11.

TYPES: BEGIN OF ty_vbap,
         vbeln TYPE vbap-vbeln,
         posnr TYPE vbap-posnr,
         matnr TYPE vbap-matnr,
         arktx TYPE vbap-arktx,
         kwmeng TYPE vbap-kwmeng,
         vrkme TYPE vbap-vrkme,
         netwr TYPE vbap-netwr,
       END OF ty_vbap.

DATA:
  gt_vbap   TYPE TABLE OF ty_vbap,
  gv_xstr   TYPE xstring,
  gt_binary TYPE solix_tab.

DATA:
  lo_document TYPE REF TO xco_cp_xlsx_document,
  lo_sheet    TYPE REF TO xco_cp_xlsx_worksheet.

*-----------------------------------------------------------------------
* Fetch Data
*-----------------------------------------------------------------------
SELECT vbeln,
       posnr,
       matnr,
       arktx,
       kwmeng,
       vrkme,
       netwr
  FROM vbap
  INTO TABLE @gt_vbap
  UP TO 10 ROWS.

IF sy-subrc <> 0.
  WRITE : 'No data found'.
  EXIT.
ENDIF.

*-----------------------------------------------------------------------
* Create XLSX
*-----------------------------------------------------------------------

lo_document = xco_cp_xlsx=>document->empty( ).

lo_sheet = lo_document->worksheet->at_position( 1 ).

"Header
lo_sheet->cell( row = 1 column = 1 )->value = 'Sales Order'.
lo_sheet->cell( row = 1 column = 2 )->value = 'Item'.
lo_sheet->cell( row = 1 column = 3 )->value = 'Material'.
lo_sheet->cell( row = 1 column = 4 )->value = 'Description'.
lo_sheet->cell( row = 1 column = 5 )->value = 'Quantity'.
lo_sheet->cell( row = 1 column = 6 )->value = 'UOM'.
lo_sheet->cell( row = 1 column = 7 )->value = 'Net Value'.

DATA(lv_row) = 2.

LOOP AT gt_vbap INTO DATA(gs_vbap).

  lo_sheet->cell( row = lv_row column = 1 )->value = gs_vbap-vbeln.
  lo_sheet->cell( row = lv_row column = 2 )->value = gs_vbap-posnr.
  lo_sheet->cell( row = lv_row column = 3 )->value = gs_vbap-matnr.
  lo_sheet->cell( row = lv_row column = 4 )->value = gs_vbap-arktx.
  lo_sheet->cell( row = lv_row column = 5 )->value = gs_vbap-kwmeng.
  lo_sheet->cell( row = lv_row column = 6 )->value = gs_vbap-vrkme.
  lo_sheet->cell( row = lv_row column = 7 )->value = gs_vbap-netwr.

  lv_row = lv_row + 1.

ENDLOOP.

*-----------------------------------------------------------------------
* Convert XLSX to XSTRING
*-----------------------------------------------------------------------

gv_xstr = lo_document->write_to_xstring( ).

*-----------------------------------------------------------------------
* Convert XSTRING -> Binary
*-----------------------------------------------------------------------

CALL FUNCTION 'SCMS_XSTRING_TO_BINARY'
  EXPORTING
    buffer     = gv_xstr
  TABLES
    binary_tab = gt_binary.

*-----------------------------------------------------------------------
* Write to AL11
*-----------------------------------------------------------------------

DATA(lv_file) = '/usr/sap/tmp/VBAP_DATA.xlsx'.

OPEN DATASET lv_file
FOR OUTPUT
IN BINARY MODE.

IF sy-subrc <> 0.
  WRITE: 'Unable to open file'.
  EXIT.
ENDIF.

LOOP AT gt_binary INTO DATA(ls_bin).
  TRANSFER ls_bin TO lv_file.
ENDLOOP.

CLOSE DATASET lv_file.

WRITE : / 'Excel file created successfully.'.
WRITE : / lv_file.
