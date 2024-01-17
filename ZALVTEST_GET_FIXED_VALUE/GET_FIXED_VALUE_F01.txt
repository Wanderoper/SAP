*&---------------------------------------------------------------------*
*&  Include           ZALVTEST_GET_FIXED_VALUE_F01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Form  GET_DATA
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM GET_DATA .
  SELECT *
    FROM SMEAL
    INTO CORRESPONDING FIELDS OF TABLE GT_ITAB
   WHERE CARRID = P_CAR.

ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  GET_FIXED_VALUE_BY_TABLE
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM GET_FIXED_VALUE_BY_TABLE .
  CLEAR: GT_TEXT, GW_TEXT. " 쓰레기값 방지

  SELECT DOMVALUE_L
         DDTEXT
    FROM DD07T
    INTO CORRESPONDING FIELDS OF TABLE GT_TEXT
   WHERE DOMNAME    = 'S_MEALTYPE'
     AND DDLANGUAGE = SY-LANGU.

ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  SET_DATA
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM SET_DATA .
  LOOP AT GT_ITAB INTO GW_ITAB.
    " 방법 1.
    READ TABLE GT_TEXT WITH KEY DOMVALUE_L = GW_ITAB-MEALTYPE
          INTO GW_TEXT.
    GW_ITAB-TEXT_OF_MEAL = GW_TEXT-DDTEXT.
    MODIFY GT_ITAB FROM GW_ITAB TRANSPORTING TEXT_OF_MEAL.

    " 방법 2.

  ENDLOOP.
ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  CREATE_ALV
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM CREATE_ALV .
  " CONTAINER
  IF SY-DYNNR = 100.
  IF GO_CONTAINER IS NOT BOUND.
    CREATE OBJECT GO_CONTAINER
      EXPORTING
        REPID                       = SY-CPROG
        DYNNR                       = SY-DYNNR
        EXTENSION                   = 2000
      EXCEPTIONS
        CNTL_ERROR                  = 1
        CNTL_SYSTEM_ERROR           = 2
        CREATE_ERROR                = 3
        LIFETIME_ERROR              = 4
        LIFETIME_DYNPRO_DYNPRO_LINK = 5
        OTHERS                      = 6
        .
    CASE SY-SUBRC.
      WHEN 0.
        " NOTHING TO DO.
      WHEN OTHERS.
        MESSAGE 'CONTAINER ERROR' TYPE 'E'.
    ENDCASE.
  ENDIF.

  " GRID
  IF GO_ALV_GRID IS NOT BOUND.
    CREATE OBJECT GO_ALV_GRID
      EXPORTING
        I_PARENT          = GO_CONTAINER
      EXCEPTIONS
        ERROR_CNTL_CREATE = 1
        ERROR_CNTL_INIT   = 2
        ERROR_CNTL_LINK   = 3
        ERROR_DP_CREATE   = 4
        OTHERS            = 5
        .
    CASE SY-SUBRC.
      WHEN 0.
        " NOTHING TO DO.
      WHEN OTHERS.
        MESSAGE 'ALV_GRID ERROR' TYPE 'E'.
    ENDCASE.
  ENDIF.

  ELSEIF SY-DYNNR = 200.
    IF GO_CONTAINER2 IS NOT BOUND.
        CREATE OBJECT GO_CONTAINER2
          EXPORTING
            REPID                       = SY-CPROG
            DYNNR                       = SY-DYNNR
            EXTENSION                   = 2000
          EXCEPTIONS
            CNTL_ERROR                  = 1
            CNTL_SYSTEM_ERROR           = 2
            CREATE_ERROR                = 3
            LIFETIME_ERROR              = 4
            LIFETIME_DYNPRO_DYNPRO_LINK = 5
            OTHERS                      = 6
            .
        CASE SY-SUBRC.
          WHEN 0.
            " NOTHING TO DO.
          WHEN OTHERS.
            MESSAGE 'CONTAINER ERROR' TYPE 'E'.
        ENDCASE.
      ENDIF.

      " GRID
      IF GO_ALV_GRID2 IS NOT BOUND.
        CREATE OBJECT GO_ALV_GRID2
          EXPORTING
            I_PARENT          = GO_CONTAINER2
          EXCEPTIONS
            ERROR_CNTL_CREATE = 1
            ERROR_CNTL_INIT   = 2
            ERROR_CNTL_LINK   = 3
            ERROR_DP_CREATE   = 4
            OTHERS            = 5
            .
        CASE SY-SUBRC.
          WHEN 0.
            " NOTHING TO DO.
          WHEN OTHERS.
            MESSAGE 'ALV_GRID ERROR' TYPE 'E'.
        ENDCASE.
      ENDIF.
  ENDIF.
ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  DISPLAY_ALV
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM DISPLAY_ALV TABLES PT_FIELDCAT  TYPE LVC_T_FCAT
                        PT_FIELDCAT2 TYPE LVC_T_FCAT.

  IF SY-DYNNR = 100.

  CALL METHOD GO_ALV_GRID->SET_TABLE_FOR_FIRST_DISPLAY
    EXPORTING
      I_STRUCTURE_NAME              = 'SMEAL'
    CHANGING
      IT_OUTTAB                     = GT_ITAB
      IT_FIELDCATALOG               = PT_FIELDCAT[]
    EXCEPTIONS
      INVALID_PARAMETER_COMBINATION = 1
      PROGRAM_ERROR                 = 2
      TOO_MANY_LINES                = 3
      OTHERS                        = 4
          .
    CASE SY-SUBRC.
      WHEN 0.
        " NOTHING TO DO.
      WHEN OTHERS.
        MESSAGE 'SET DISPLAY ERROR' TYPE 'E'.
    ENDCASE.

    ELSEIF SY-DYNNR = 200.
      CALL METHOD GO_ALV_GRID2->SET_TABLE_FOR_FIRST_DISPLAY
        EXPORTING
*          I_BUFFER_ACTIVE               =
*          I_BYPASSING_BUFFER            =
*          I_CONSISTENCY_CHECK           =
          I_STRUCTURE_NAME              = 'SMEAL'
*          IS_VARIANT                    =
*          I_SAVE                        =
*          I_DEFAULT                     = 'X'
*          IS_LAYOUT                     =
*          IS_PRINT                      =
*          IT_SPECIAL_GROUPS             =
*          IT_TOOLBAR_EXCLUDING          =
*          IT_HYPERLINK                  =
*          IT_ALV_GRAPHICS               =
*          IT_EXCEPT_QINFO               =
*          IR_SALV_ADAPTER               =
        CHANGING
          IT_OUTTAB                     = GT_MEAL
          IT_FIELDCATALOG               = PT_FIELDCAT2[]
*          IT_SORT                       =
*          IT_FILTER                     =
        EXCEPTIONS
          INVALID_PARAMETER_COMBINATION = 1
          PROGRAM_ERROR                 = 2
          TOO_MANY_LINES                = 3
          OTHERS                        = 4
              .
        CASE SY-SUBRC.
          WHEN 0.
            " NOTHING TO DO.
          WHEN OTHERS.
            MESSAGE 'SET DISPLAY ERROR' TYPE 'E'.
        ENDCASE.
    ENDIF.
ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  SET_FIELDCAT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      -->P_LT_FIELDCAT  text
*----------------------------------------------------------------------*
FORM SET_FIELDCAT  TABLES   PT_FIELDCAT  TYPE LVC_T_FCAT
                            PT_FIELDCAT2 TYPE LVC_T_FCAT.
  DATA: LW_FIELDCAT  TYPE LVC_S_FCAT,
        LW_FIELDCAT2 TYPE LVC_S_FCAT.

  IF SY-DYNNR = 100.

    " CARRID, MEALNUMBER, MEALTYPE
    CLEAR  LW_FIELDCAT.
    LW_FIELDCAT = VALUE #( FIELDNAME = 'CARRID'
                           COLTEXT   = '항공사'
                           COL_POS   = 1
                           NO_OUT    = ''       ).
    APPEND LW_FIELDCAT TO PT_FIELDCAT.

    CLEAR  LW_FIELDCAT.
    LW_FIELDCAT = VALUE #( FIELDNAME = 'MEALNUMBER'
                           COLTEXT   = '일련번호'
                           COL_POS   = 2
                           NO_OUT    = ''       ).
    APPEND LW_FIELDCAT TO PT_FIELDCAT.

    CLEAR  LW_FIELDCAT.
    LW_FIELDCAT = VALUE #( FIELDNAME = 'MEALTYPE'
                           COLTEXT   = '식사종류'
*                           COL_POS   = 3
                           NO_OUT    = 'X'       ).
    APPEND LW_FIELDCAT TO PT_FIELDCAT.

    CLEAR  LW_FIELDCAT.
    LW_FIELDCAT = VALUE #( FIELDNAME = 'TEXT_OF_MEAL'
                           COLTEXT   = '식사'
                           COL_POS   = 3
                           NO_OUT    = ''       ).
    APPEND LW_FIELDCAT TO PT_FIELDCAT.

  ELSEIF SY-DYNNR = 200.

    " CARRID, MEALNUMBER, MEALTYPE
    CLEAR  LW_FIELDCAT2.
    LW_FIELDCAT2 = VALUE #( FIELDNAME = 'CARRID'
                           COLTEXT   = '항공사'
                           COL_POS   = 2
                           NO_OUT    = ''       ).
    APPEND LW_FIELDCAT2 TO PT_FIELDCAT2.

    CLEAR  LW_FIELDCAT2.
    LW_FIELDCAT2 = VALUE #( FIELDNAME = 'MEALNUMBER'
                           COLTEXT   = '일련번호'
                           COL_POS   = 3
                           NO_OUT    = ''       ).
    APPEND LW_FIELDCAT2 TO PT_FIELDCAT2.

    CLEAR  LW_FIELDCAT2.
    LW_FIELDCAT2 = VALUE #( FIELDNAME = 'MEALTYPE'
                           COLTEXT   = '식사종류'
                           COL_POS   = 1
                           NO_OUT    = ''       ).
    APPEND LW_FIELDCAT2 TO PT_FIELDCAT2.
  ENDIF.
ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  FREE_RESOURCES
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM FREE_RESOURCES .
  IF SY-DYNNR = 100.
    FREE:  GO_CONTAINER,
           GO_ALV_GRID.
    CLEAR: OK_CODE.

  ELSEIF SY-DYNNR = 200.
    FREE:  GO_CONTAINER2,
           GO_ALV_GRID2.
    CLEAR: OK_CODE.

  ENDIF.

ENDFORM.