*&---------------------------------------------------------------------*
*&  Include           ZB10_PROGRAMPBO
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Module  STATUS_0100  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE STATUS_0100 OUTPUT.
  CLEAR OK_CODE.
  PERFORM STATUS_0100.
ENDMODULE.
*&---------------------------------------------------------------------*
*&      Module  ALV_GRID_0100  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE ALV_GRID_0100 OUTPUT.
    DATA: LT_FIELDCAT TYPE LVC_T_FCAT  ,
          LS_LAYOUT   TYPE LVC_S_LAYO  ,
          LT_EXCLUDED TYPE UI_FUNCTIONS.

    PERFORM: SET_CONTAINER                     ,
             SET_GRID                          ,
             SET_STYLE                         ,
             SET_ALV_LAYOUT  CHANGING LS_LAYOUT,
             SET_ALV_TOOLBAR TABLES LT_EXCLUDED,
             SET_FIELDCAT    TABLES LT_FIELDCAT,
             SET_EVENT                         ,
             DISPLAY_ALV     TABLES LT_FIELDCAT
                                    LT_EXCLUDED
                             USING  LS_LAYOUT  .

ENDMODULE.