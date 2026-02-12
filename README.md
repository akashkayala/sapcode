AT SELECTION-SCREEN ON VALUE-REQUEST FOR p_file.
  CALL FUNCTION 'F4_FILENAME'
    IMPORTING
      file_name = p_file.  



  CREATE DATA lt_dref TYPE TABLE OF (p_table).
  CREATE DATA ls_dref TYPE (p_table).

  ASSIGN lt_dref->* TO <ft_table>.

  ASSIGN ls_dref->* TO <fs_table>.

  CALL FUNCTION 'ALSM_EXCEL_TO_INTERNAL_TABLE'
    EXPORTING
      filename                = p_file
      i_begin_col             = 1
      i_begin_row             = 1
      i_end_col               = 99
      i_end_row               = 999999
    TABLES
      intern                  = lt_excel
    EXCEPTIONS
      inconsistent_parameters = 1
      upload_ole              = 2
      OTHERS                  = 3.
  IF sy-subrc EQ 0.
    SORT lt_excel BY row.
 LOOP AT lt_excel INTO DATA(ls_excel).

   lv_col = ls_excel-col + 1.
      ASSIGN COMPONENT lv_col OF STRUCTURE <fs_table> TO <dyn_field>.
      IF sy-subrc = 0.

            IF ls_excel-row = 1.
              CALL FUNCTION 'DDIF_FIELDINFO_GET'
                EXPORTING
                  tabname        = p_table
                TABLES
                  dfies_tab      = lt_table_filds
                EXCEPTIONS
                  not_found      = 1
                  internal_error = 2
                  OTHERS         = 3.
              READ TABLE lt_table_filds INTO DATA(ls_table_fields) INDEX lv_col.
              IF sy-subrc = 0.
                IF ls_table_fields-fieldname NE ls_excel-value.
                  WRITE: 'Excel sheet data and Dynamic table fields are not matching'.
                  EXIT.
                ENDIF.
              ENDIF.
            ELSE.
              <dyn_field> = ls_excel-value.
            ENDIF.
      ENDIF.
      IF ls_excel-row GT 1.
        AT END OF row.
          APPEND <fs_table> TO <ft_table>.
          CLEAR <fs_table>.
        ENDAT.
      ENDIF.
    ENDLOOP.

    IF <ft_table> IS NOT INITIAL.
      IF p_test IS INITIAL.

        MODIFY (p_table) FROM TABLE <ft_table>.
        IF sy-subrc EQ 0.
          COMMIT WORK.
          DATA(lv_lines) = lines( <ft_table> ).
          WRITE: TEXT-002,lv_lines.
        ELSE.
          ROLLBACK WORK.
          MESSAGE TEXT-003 TYPE 'E' DISPLAY LIKE 'I'.
        ENDIF.
      ELSE.

        cl_salv_table=>factory(
          IMPORTING
            r_salv_table   =  lo_alv
          CHANGING
            t_table        = <ft_table>
        ).

        lo_alv->display( ).
      ENDIF.
    ENDIF.
  ENDIF.
