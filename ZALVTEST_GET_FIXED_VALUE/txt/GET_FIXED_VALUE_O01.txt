*&---------------------------------------------------------------------*
*&  Include           ZALVTEST_GET_FIXED_VALUE_O01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Module  STATUS_0100  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE STATUS_0100 OUTPUT.
  SET PF-STATUS 'G0100'.
  SET TITLEBAR 'T0100' WITH TEXT-T01.
ENDMODULE.
*&---------------------------------------------------------------------*
*&      Module  SET_ALV  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE SET_ALV OUTPUT.
  DATA: LT_FIELDCAT TYPE LVC_T_FCAT.
  DATA: LT_FIELDCAT2 TYPE LVC_T_FCAT.

  PERFORM: CREATE_ALV,
           SET_FIELDCAT TABLES LT_FIELDCAT LT_FIELDCAT2,
           SET_EVENT    ,
           DISPLAY_ALV  TABLES LT_FIELDCAT LT_FIELDCAT2.
ENDMODULE.
*&---------------------------------------------------------------------*
*&      Module  STATUS_0200  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE STATUS_0200 OUTPUT.
  SET PF-STATUS 'G0200'.
  SET TITLEBAR 'T0200' WITH TEXT-T02 TITLE_200.
  CLEAR: TITLE_200.
ENDMODULE.
*&---------------------------------------------------------------------*
*&      Form  SET_EVENT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM SET_EVENT .

  " EVENT HANDLER
  IF GO_EVENT_RECEIVER IS NOT BOUND.
    CREATE OBJECT GO_EVENT_RECEIVER.
    SET HANDLER GO_EVENT_RECEIVER->HANDLE_DOUBLE_CLICK FOR GO_ALV_GRID.
  ENDIF.

ENDFORM.