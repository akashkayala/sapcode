PARAMETERS: p_struct TYPE tabname DEFAULT 'ACDOCA',
            p_file   TYPE string  DEFAULT 'C:\temp\Template.xlsx'.

DATA: lo_struct     TYPE REF TO cl_abap_structdescr,
      lo_table_desc TYPE REF TO cl_abap_tabledescr,
      lr_data       TYPE REF TO data,
      lo_salv       TYPE REF TO cl_salv_table,
      lv_xml        TYPE xstring,
      lt_binary_tab TYPE solix_tab,
      lv_length     TYPE i.

FIELD-SYMBOLS: <lt_template> TYPE STANDARD TABLE.

START-OF-SELECTION.

  " 1. Get the Structure Definition from SE11
  lo_struct ?= cl_abap_typedescr=>describe_by_name( p_struct ).

  " 2. Create Dynamic Table and Assign to Field Symbol
  lo_table_desc = cl_abap_tabledescr=>create( lo_struct ).
  CREATE DATA lr_data TYPE HANDLE lo_table_desc.
  ASSIGN lr_data->* TO <lt_template>.

  " 3. Add an initial row for the template
  APPEND INITIAL LINE TO <lt_template>.

  TRY.
      " 4. Create SALV Instance (invisible, just for conversion)
      cl_salv_table=>factory(
        IMPORTING r_salv_table = lo_salv
        CHANGING  t_table      = <lt_template> ).

      " Set technical headers (identical to original logic)
      DATA(lo_cols) = lo_salv->get_columns( ).
      DATA(lt_cols) = lo_cols->get( ).
      LOOP AT lt_cols INTO DATA(ls_col).
        ls_col-r_column->set_short_text( |{ ls_col-columnname }| ).
        ls_col-r_column->set_medium_text( |{ ls_col-columnname }| ).
        ls_col-r_column->set_long_text( |{ ls_col-columnname }| ).
      ENDLOOP.

      " 5. Convert to XLSX XML Format
      lv_xml = lo_salv->to_xml( if_salv_bs_xml=>c_type_xlsx ).

      " 6. Convert XString to Binary Table
      CALL FUNCTION 'SCMS_XSTRING_TO_BINARY'
        EXPORTING
          buffer          = lv_xml
        IMPORTING
          output_length   = lv_length
        TABLES
          binary_tab      = lt_binary_tab.

      " 7. Download to Local System
      cl_gui_frontend_services=>gui_download(
        EXPORTING
          bin_filesize = lv_length
          filename     = p_file
          filetype     = 'BIN'
        CHANGING
          data_tab     = lt_binary_tab
        EXCEPTIONS
          OTHERS       = 1 ).

      IF sy-subrc = 0.
        MESSAGE 'XLSX Template created successfully!' TYPE 'S'.
      ENDIF.

    CATCH cx_root.
      MESSAGE 'Error generating XLSX.' TYPE 'E'.
  ENDTRY.
