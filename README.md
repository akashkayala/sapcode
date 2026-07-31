REPORT z_vbap_to_xlsx_al11.

PARAMETERS: p_file TYPE rlgrap-filename LOWER CASE
            DEFAULT '/usr/sap/tmp/vbap_10_rows.xlsx'.

TYPES: BEGIN OF ty_vbap,
         vbeln  TYPE vbap-vbeln,
         posnr  TYPE vbap-posnr,
         matnr  TYPE vbap-matnr,
         arktx  TYPE vbap-arktx,
         kwmeng TYPE vbap-kwmeng,
         vrkme  TYPE vbap-vrkme,
         netwr  TYPE vbap-netwr,
       END OF ty_vbap.

DATA: gt_vbap   TYPE STANDARD TABLE OF ty_vbap,
      gt_fcat   TYPE lvc_t_fcat,
      gs_fcat   TYPE lvc_s_fcat,
      gv_xlsx   TYPE xstring,
      gr_data   TYPE REF TO data,
      gr_result TYPE REF TO cl_salv_ex_result_data_table,
      go_error  TYPE REF TO cx_root.

START-OF-SELECTION.

  SELECT vbeln
         posnr
         matnr
         arktx
         kwmeng
         vrkme
         netwr
    FROM vbap
    INTO TABLE gt_vbap
    UP TO 10 ROWS.

  IF gt_vbap IS INITIAL.
    MESSAGE 'No data found in VBAP table' TYPE 'I'.
    RETURN.
  ENDIF.

  PERFORM build_fieldcatalog CHANGING gt_fcat.

  GET REFERENCE OF gt_vbap INTO gr_data.

  TRY.

      gr_result = cl_salv_ex_util=>factory_result_data_table(
                    r_data         = gr_data
                    t_fieldcatalog = gt_fcat ).

      cl_salv_bs_tt_util=>if_salv_bs_tt_util~transform(
        EXPORTING
          xml_type      = if_salv_bs_xml=>c_type_xlsx
          xml_version   = cl_salv_bs_a_xml_base=>get_version( )
          r_result_data = gr_result
          xml_flavour   = if_salv_bs_c_tt=>c_tt_xml_flavour_export
          gui_type      = if_salv_bs_xml=>c_gui_type_gui
        IMPORTING
          xml           = gv_xlsx ).

    CATCH cx_root INTO go_error.
      MESSAGE go_error->get_text( ) TYPE 'E'.
  ENDTRY.

  PERFORM write_file_to_al11 USING p_file gv_xlsx.

FORM build_fieldcatalog CHANGING ct_fcat TYPE lvc_t_fcat.

  DEFINE add_field.
    CLEAR gs_fcat.
    gs_fcat-fieldname = &1.
    gs_fcat-coltext   = &2.
    gs_fcat-scrtext_l = &2.
    gs_fcat-scrtext_m = &2.
    gs_fcat-scrtext_s = &2.
    gs_fcat-outputlen = &3.
    APPEND gs_fcat TO ct_fcat.
  END-OF-DEFINITION.

  add_field 'VBELN'  'Sales Document' 10.
  add_field 'POSNR'  'Item'           6.
  add_field 'MATNR'  'Material'       18.
  add_field 'ARKTX'  'Description'    40.
  add_field 'KWMENG' 'Order Qty'      15.
  add_field 'VRKME'  'Sales Unit'     5.
  add_field 'NETWR'  'Net Value'      15.

ENDFORM.

FORM write_file_to_al11 USING iv_file TYPE rlgrap-filename
                              iv_xlsx TYPE xstring.

  OPEN DATASET iv_file FOR OUTPUT IN BINARY MODE.

  IF sy-subrc <> 0.
    MESSAGE |Unable to open AL11 file path: { iv_file }| TYPE 'E'.
  ENDIF.

  TRANSFER iv_xlsx TO iv_file.

  IF sy-subrc <> 0.
    CLOSE DATASET iv_file.
    MESSAGE |Unable to write XLSX file to AL11: { iv_file }| TYPE 'E'.
  ENDIF.

  CLOSE DATASET iv_file.

  MESSAGE |XLSX file created successfully in AL11: { iv_file }| TYPE 'S'.

ENDFORM.
