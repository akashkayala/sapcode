FORM send_mail_with_table.
  lv_html_body = '<html><body><h3>Flight Details:</h3>'.
  lv_html_body = |{ lv_html_body }<table border="1" cellpadding="5" cellspacing="0">|.
  lv_html_body = |{ lv_html_body }<tr><th>File Created</th><th>Company Code</th><th>Document Type</th><th>Currency</th><th>Sum of Debit</th><th>Sum of Credit</th><th>Document No</th><th>Success/Fail</th></tr>|.

  LOOP AT lt_table ASSIGNING FIELD-SYMBOL(<fs_lt_table>).
    lv_html_body = |{ lv_html_body }<tr>|.
    lv_html_body = |{ lv_html_body }<td>{ <fs_lt_table>-carrid }</td>|.
    lv_html_body = |{ lv_html_body }<td>{ <fs_lt_table>-connid }</td>|.
    lv_html_body = |{ lv_html_body }<td>{ <fs_lt_table>-fldate DATE = USER }</td>|.
    lv_html_body = |{ lv_html_body }<td>{ <fs_lt_table>-price }</td>|.
    lv_html_body = |{ lv_html_body }</tr>|.
  ENDLOOP.

  lv_html_body = |{ lv_html_body }</table></body></html>|.

  DATA(lt_soli) = cl_bcs_convert=>string_to_soli( iv_string = lv_html_body ).
  DATA(lo_document) = cl_document_bcs=>create_document(
      i_type    = 'HTM'
      i_text    = lt_soli
      i_subject = 'Dynamic Flight Information' ).
  DATA(lo_sender)    = cl_sapuser_bcs=>create( sy-uname ).
  DATA(lo_recipient) = cl_cam_address_bcs=>create_internet_address( 'pmeruva@publicstorage.com' ).
  DATA(lo_send_request) = cl_bcs=>create_persistent( ).
  lo_send_request->set_document( lo_document ).
  lo_send_request->set_sender( i_sender = lo_sender ).
  lo_send_request->add_recipient( i_recipient = lo_recipient ).
  lo_send_request->set_send_immediately( abap_true ). " Send immediately
  DATA(lv_sent_to_all) = lo_send_request->send( ).

  IF lv_sent_to_all = abap_true.
    COMMIT WORK AND WAIT.
  ENDIF.
ENDFORM.
