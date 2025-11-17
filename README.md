*&---------------------------------------------------------------------*
*& Report ZPROGAM_STOCK
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT zprogam_stock.
TABLES: marc.
TYPES: BEGIN OF ty_marc,
         matnr TYPE matnr,
         werks TYPE werks,
       END OF ty_marc.
TYPES: BEGIN OF ty_mdps.
         INCLUDE STRUCTURE mdps.
TYPES: END OF ty_mdps.
TYPES: BEGIN OF ty_mdez.
         INCLUDE STRUCTURE mdez.
TYPES: END OF ty_mdez.
TYPES: BEGIN OF ty_mdsu.
         INCLUDE STRUCTURE mdsu.
TYPES: END OF ty_mdsu.
TYPES: BEGIN OF lty_mdps,
         plaab  TYPE plaab,
         ematn  TYPE ematn,
         "imwerk TYPE imwerk,
       END OF lty_mdps.
DATA: it_mdps TYPE STANDARD TABLE OF lty_mdps,
      wa_mdps TYPE lty_mdps.

DATA: gt_mdps TYPE STANDARD TABLE OF ty_mdps,
      gt_mdez TYPE STANDARD TABLE OF ty_mdez,
      gt_mdsu TYPE STANDARD TABLE OF ty_mdsu.


DATA: it_marc TYPE STANDARD  TABLE OF ty_marc,
      wa_marc TYPE ty_marc.

SELECTION-SCREEN BEGIN OF BLOCK b1 WITH FRAME TITLE TEXT-003 .
  SELECT-OPTIONS : s_matnr FOR marc-matnr,
                  s_werks FOR marc-werks.
  PARAMETERS: p_single TYPE char01 RADIOBUTTON GROUP r1,
              p_clctv  TYPE char01 RADIOBUTTON GROUP r1.
SELECTION-SCREEN END OF BLOCK b1.

START-OF-SELECTION.

  PERFORM get_md04_data.

  IF p_single = 'X'.
    PERFORM display_single_alv.
  ENDIF.

  IF p_clctv = 'X'.
    PERFORM display_collective_alv.
  ENDIF.

FORM get_md04_data.

  SELECT matnr, werks
    FROM marc
    INTO TABLE @it_marc
                    WHERE matnr IN @s_matnr
                    AND werks IN @s_werks.

  IF it_marc IS INITIAL.
    MESSAGE 'No MARC data found' TYPE 'E'.
  ENDIF.

  IF it_marc IS NOT INITIAL.
    LOOP AT it_marc INTO wa_marc.
      CLEAR: gt_mdps, gt_mdez, gt_mdsu.
      CALL FUNCTION 'MD_STOCK_REQUIREMENTS_LIST_API'
        EXPORTING
*         PLSCN                    =
          matnr                    = wa_marc-matnr
          werks                    = wa_marc-werks
*         BERID                    =
*         ERGBZ                    =
*         AFIBZ                    =
*         INPER                    =
*         DISPLAY_LIST_MDPSX       =
*         DISPLAY_LIST_MDEZX       =
*         DISPLAY_LIST_MDSUX       =
*         NOBUF                    =
*         PLAUF                    =
*         I_VRFWE                  =
*         IS_SFILT                 =
*         IS_AFILT                 =
*         IV_FILL_MDSTA            = 'X'
* IMPORTING
*         E_MT61D                  =
*         E_MDKP                   =
*         E_CM61M                  =
*         E_MDSTA                  =
*         E_ERGBZ                  =
*         E_CM61W                  =
        TABLES
          mdpsx                    = gt_mdps
          mdezx                    = gt_mdez
          mdsux                    = gt_mdsu
        EXCEPTIONS
          material_plant_not_found = 1
          plant_not_found          = 2
          OTHERS                   = 3.
      IF sy-subrc <> 0.
* Implement suitable error handling here

      ENDIF.
      LOOP AT gt_mdps ASSIGNING FIELD-SYMBOL(<fs_mdps>).
        <fs_mdps>-ematn = wa_mdps-ematn.
        <fs_mdps>-plaab = wa_mdps-plaab.
      ENDLOOP.
    ENDLOOP.
  ENDIF.

ENDFORM.

FORM display_single_alv.

  DATA: lo_alv TYPE REF TO cl_salv_table.

  TRY.
      CALL METHOD cl_salv_table=>factory
        IMPORTING
          r_salv_table = lo_alv
        CHANGING
          t_table      = gt_mdps.

      lo_alv->get_columns( )->set_optimize( abap_true ).
      lo_alv->display( ).

    CATCH cx_salv_msg INTO DATA(lx_msg).
      MESSAGE lx_msg->get_text( ) TYPE 'E'.
  ENDTRY.

ENDFORM.

FORM display_collective_alv.

  "Combine several MDPS records of all materials
  DATA lt_all TYPE STANDARD TABLE OF ty_mdps.

  APPEND LINES OF gt_mdps TO lt_all.
  DATA: lo_alv2 TYPE REF TO cl_salv_table.

*  DATA(lo_alv2) = NEW cl_salv_table( ).


  TRY.
      CALL METHOD cl_salv_table=>factory
        IMPORTING
          r_salv_table = lo_alv2
        CHANGING
          t_table      = lt_all.

      lo_alv2->get_columns( )->set_optimize( 'X' ).
      lo_alv2->display( ).

    CATCH cx_salv_msg INTO DATA(lx_msg2).
      MESSAGE lx_msg2->get_text( ) TYPE 'E'.
  ENDTRY.

ENDFORM.
