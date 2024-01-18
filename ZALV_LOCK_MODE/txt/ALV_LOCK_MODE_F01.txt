*&---------------------------------------------------------------------*
*&  Include           ZB10_PROGRAMF01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Form  INITIALIZATION
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM INITIALIZATION .
  SO_CAR-LOW = 'AA'.
  APPEND SO_CAR.

  GV_EDIT = 0. " 처음 1000번 화면 시작 시 READ MODE (GV_EDIT = 0 조회 1 수정모드) 를 활성화해준다
ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  GET_DATA
*&---------------------------------------------------------------------*
*       foreign lock 이 잡힌 경우 gv_read_table = 1 
*       내가 lock을 잡는 경우 gv_read_table = 0
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM GET_DATA .
  PERFORM SET_LOCK.

  IF GV_READ_TABLE = ABAP_TRUE.   " foreign lock이 이미 있어서 조회모드로 들어온 경우는 수정이 불가함
    SELECT *
      FROM ZB10_SFLIGHT_TES
      INTO CORRESPONDING FIELDS OF TABLE GT_SFLIGHT
     WHERE CARRID IN SO_CAR.

     IF GT_SFLIGHT IS INITIAL.
       MESSAGE E001 WITH TEXT-E01.
     ELSE.
       CALL SCREEN 0100.  " call screen 을 여기가 아니라 end-of-selection에서 하자
     ENDIF.
  ELSE.
    SELECT *
      FROM ZB10_SFLIGHT_TES
      INTO CORRESPONDING FIELDS OF TABLE GT_BUFFER. " READ 읽기 -> 검증용

    SELECT *
      FROM ZB10_SFLIGHT_TES
      INTO CORRESPONDING FIELDS OF TABLE GT_SFLIGHT
     WHERE CARRID IN SO_CAR.

     IF GT_SFLIGHT IS INITIAL.
       MESSAGE E001 WITH TEXT-E01.
     ELSE.
       CALL SCREEN 0100.
     ENDIF.
   ENDIF.
ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  SET_CONTAINER
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM SET_CONTAINER .
  IF GO_DOCKING IS NOT BOUND.
  CREATE OBJECT GO_DOCKING
      EXPORTING
*        PARENT                      =
        REPID                       = SY-REPID
        DYNNR                       = SY-DYNNR
*        SIDE                        = DOCK_AT_LEFT
        EXTENSION                   = 3000
*        STYLE                       =
*        LIFETIME                    = lifetime_default
*        CAPTION                     =
*        METRIC                      = 0
*        RATIO                       = " 5-95
*        NO_AUTODEF_PROGID_DYNNR     =
*        NAME                        =
*      EXCEPTIONS
*        CNTL_ERROR                  = 1
*        CNTL_SYSTEM_ERROR           = 2
*        CREATE_ERROR                = 3
*        LIFETIME_ERROR              = 4
*        LIFETIME_DYNPRO_DYNPRO_LINK = 5
*        OTHERS                      = 6
        .
    IF SY-SUBRC <> 0.
      MESSAGE A001 WITH 'Failed to load Container Object'(E02).
    ENDIF.
  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  SET_GRID
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM SET_GRID .
  IF GO_GRID IS NOT BOUND.
    CREATE OBJECT GO_GRID
      EXPORTING
*        I_SHELLSTYLE      = 0
*        I_LIFETIME        =
        I_PARENT          = GO_DOCKING
*        I_APPL_EVENTS     = space
*        I_PARENTDBG       =
*        I_APPLOGPARENT    =
*        I_GRAPHICSPARENT  =
*        I_NAME            =
*        I_FCAT_COMPLETE   = SPACE
*      EXCEPTIONS
*        ERROR_CNTL_CREATE = 1
*        ERROR_CNTL_INIT   = 2
*        ERROR_CNTL_LINK   = 3
*        ERROR_DP_CREATE   = 4
*        OTHERS            = 5
        .
    IF SY-SUBRC <> 0.
      MESSAGE A001 WITH 'Failed to load the Grid'(E03).
    ENDIF.

  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  FREE_CONTROL_RESOURCES
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM FREE_CONTROL_RESOURCES .
  CALL METHOD GO_GRID->FREE.
  CALL METHOD GO_DOCKING->FREE.
  FREE: GO_GRID, GO_DOCKING.

  PERFORM FREE_LOCK.
  CLEAR: GV_READ_TABLE.
ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  DISPLAY_ALV
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM DISPLAY_ALV TABLES PT_FIELDCAT     TYPE LVC_T_FCAT
                        PT_EXCLUDED     TYPE UI_FUNCTIONS
                 USING VALUE(PS_LAYOUT) TYPE LVC_S_LAYO.

*  PERFORM SET_LAYOUT.
  DATA(LS_VARIANT) = VALUE DISVARIANT( REPORT = SY-CPROG VARIANT = P_VAR ).

  CALL METHOD GO_GRID->SET_TABLE_FOR_FIRST_DISPLAY
    EXPORTING
*      I_BUFFER_ACTIVE               =
*      I_BYPASSING_BUFFER            =
*      I_CONSISTENCY_CHECK           =
      I_STRUCTURE_NAME              = 'ZB10_SFLIGHT_TES'
      IS_VARIANT                    = LS_VARIANT
      I_SAVE                        = 'A'
*      I_DEFAULT                     = 'X'
      IS_LAYOUT                     = PS_LAYOUT
*      IS_PRINT                      =
*      IT_SPECIAL_GROUPS             =
*      IT_TOOLBAR_EXCLUDING          = PT_EXCLUDED[]
*      IT_HYPERLINK                  =
*      IT_ALV_GRAPHICS               =
*      IT_EXCEPT_QINFO               =
*      IR_SALV_ADAPTER               =
    CHANGING
      IT_OUTTAB                     = GT_SFLIGHT
      IT_FIELDCATALOG               = PT_FIELDCAT[]
*      IT_SORT                       =
*      IT_FILTER                     =
    EXCEPTIONS
*      INVALID_PARAMETER_COMBINATION = 1
*      PROGRAM_ERROR                 = 2
*      TOO_MANY_LINES                = 3
      OTHERS                        = 4
          .
  IF SY-SUBRC <> 0.
    MESSAGE A001 WITH 'Error displaying table'(E04).
  ENDIF.

  CALL METHOD GO_GRID->SET_READY_FOR_INPUT
    EXPORTING
      I_READY_FOR_INPUT    = GV_EDIT.


ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  REFRESH_ALV
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM REFRESH_ALV.
  DATA(LS_STABLE) = VALUE LVC_S_STBL( ROW = 'X' COL = 'X' ).

  CALL METHOD GO_GRID->REFRESH_TABLE_DISPLAY
    EXPORTING
      IS_STABLE      = LS_STABLE
*      I_SOFT_REFRESH =
*    EXCEPTIONS
*      FINISHED       = 1
*      OTHERS         = 2
          .
  IF SY-SUBRC <> 0.
*   Implement suitable error handling here
  ENDIF.


ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  SET_ALV_LAYOUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM SET_ALV_LAYOUT CHANGING PS_LAYOUT TYPE LVC_S_LAYO.

  CLEAR PS_LAYOUT.

  PS_LAYOUT = VALUE #( GRID_TITLE  = SY-CPROG
                       ZEBRA       = 'X'
*                       NO_HEADERS  =
                       SEL_MODE    = 'A'
                       CWIDTH_OPT  = 'X'
*                       NO_TOOLBAR  =
*                       TOTALS_BEF  =
*                       INFO_FNAME  = 'LINECOLOR'
*                       CTAB_FNAME  = 'COLCOLOR'
                       EDIT        = ''
*                       NO_ROWINS   =
                       STYLEFNAME  = 'CELLTAB'
                                                    ).


ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  SET_FIELDCAT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      -->P_LT_FIELDCAT  text
*----------------------------------------------------------------------*
FORM SET_FIELDCAT  TABLES   PT_FIELDCAT TYPE LVC_T_FCAT.
  DATA: LW_FIELDCAT TYPE LVC_S_FCAT.

*  CLEAR LW_FIELDCAT.
*  LW_FIELDCAT = VALUE #( FIELDNAME = 'FLDATE'
*                         EDIT      = 'X'       ).
*  MODIFY TABLE PT_FIELDCAT FROM LW_FIELDCAT.
*
*  CLEAR LW_FIELDCAT.
*  LW_FIELDCAT = VALUE #( FIELDNAME = 'PRICE'
*                         EDIT      = 'X'       ).
*  MODIFY TABLE PT_FIELDCAT FROM LW_FIELDCAT.
*
*  CLEAR LW_FIELDCAT.
*  LW_FIELDCAT = VALUE #( FIELDNAME = 'SEATSOCC'
*                         EDIT      = 'X'       ).
*  MODIFY TABLE PT_FIELDCAT FROM LW_FIELDCAT.

ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  SET_EVENT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM SET_EVENT .

  SET HANDLER LCL_EVENT_HANDLER=>ON_DOUBLE_CLICK FOR GO_GRID.

ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  SET_ALV_TOOLBAR
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM SET_ALV_TOOLBAR TABLES PT_EXCLUDED TYPE UI_FUNCTIONS.

  APPEND: CL_GUI_ALV_GRID=>MC_FC_FILTER TO PT_EXCLUDED.

ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  SET_STYLE
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM SET_STYLE .
  DATA LS_CELLTAB TYPE LVC_S_STYL.

  IF GV_EDIT = 1.
    LOOP AT GT_SFLIGHT INTO GW_SFLIGHT.
      IF GW_SFLIGHT-FLDATE < SY-DATUM.
        CLEAR LS_CELLTAB.
        LS_CELLTAB = VALUE #( FIELDNAME = 'FLDATE' STYLE = CL_GUI_ALV_GRID=>MC_STYLE_DISABLED ).
        INSERT LS_CELLTAB INTO TABLE GW_SFLIGHT-CELLTAB.

        CLEAR LS_CELLTAB.
        LS_CELLTAB = VALUE #( FIELDNAME = 'PRICE' STYLE = CL_GUI_ALV_GRID=>MC_STYLE_ENABLED ).
        INSERT LS_CELLTAB INTO TABLE GW_SFLIGHT-CELLTAB.

      ELSE.
        CLEAR LS_CELLTAB.
        LS_CELLTAB = VALUE #( FIELDNAME = 'FLDATE' STYLE = CL_GUI_ALV_GRID=>MC_STYLE_ENABLED ).
        INSERT LS_CELLTAB INTO TABLE GW_SFLIGHT-CELLTAB.

        CLEAR LS_CELLTAB.
        LS_CELLTAB = VALUE #( FIELDNAME = 'PRICE' STYLE = CL_GUI_ALV_GRID=>MC_STYLE_ENABLED ).
        INSERT LS_CELLTAB INTO TABLE GW_SFLIGHT-CELLTAB.

      ENDIF.

      MODIFY GT_SFLIGHT FROM GW_SFLIGHT TRANSPORTING CELLTAB.
    ENDLOOP.
  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  SWITCH_EDIT_MODE
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM SWITCH_EDIT_MODE .

**  IF GO_GRID->IS_READY_FOR_INPUT( ) EQ 0.
*  IF GV_EDIT EQ 0.
** SET EDIT & LOCK EDIT
    CALL METHOD GO_GRID->SET_READY_FOR_INPUT
                     EXPORTING I_READY_FOR_INPUT = COND #( LET EDIT = 0 READ = 1 IN
                                                           WHEN GV_EDIT = 0 THEN 1
                                                           WHEN GV_EDIT = 1 THEN 0 ).
    IF GV_EDIT = 0.
      GV_EDIT = 1.
    ELSE.
      GV_EDIT = 0.
    ENDIF.

**    GV_EDIT = 1.
*  ELSE.
** lock edit
*
*    CALL METHOD GO_GRID->SET_READY_FOR_INPUT
*                    EXPORTING I_READY_FOR_INPUT = GV_EDIT.
**    GV_EDIT = 0.
*  ENDIF.
ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  LOCK
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM SET_LOCK .
  DATA LV_ANSWER TYPE C.

  CALL FUNCTION 'ENQUEUE_EY_SFLIGHT_TES'
    EXPORTING
*      MODE_ZB10_SFLIGHT_TES       = 'E'
      MANDT                       = SY-MANDT
      CARRID                      = SO_CAR-LOW
*      CONNID                      =
*      FLDATE                      =
*      X_CARRID                    = ' '
*      X_CONNID                    = ' '
*      X_FLDATE                    = ' '
*      _SCOPE                      = '2'
*      _WAIT                       = ' '
*      _COLLECT                    = ' '
    EXCEPTIONS
      FOREIGN_LOCK                = 1
      SYSTEM_FAILURE              = 2
      OTHERS                      = 3
            .
  CASE SY-SUBRC.
    WHEN 0.
      " SUCCESS
    WHEN 1.
      GV_READ_TABLE = ABAP_TRUE.
      CONFIRM TEXT-C01 TEXT-C02 LV_ANSWER.
      CASE LV_ANSWER.
        WHEN 1.
          " NOTHING TO DO
        WHEN 2.
          MESSAGE I010.
        WHEN 'A'.
          MESSAGE I010.
      ENDCASE.
  ENDCASE.

ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  FREE_LOCK
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM FREE_LOCK .
  CALL FUNCTION 'DEQUEUE_EY_SFLIGHT_TES'
   EXPORTING
*     MODE_ZB10_SFLIGHT_TES       = 'E'
     MANDT                       = SY-MANDT
     CARRID                      = SO_CAR-LOW
*     CONNID                      =
*     FLDATE                      =
*     X_CARRID                    = ' '
*     X_CONNID                    = ' '
*     X_FLDATE                    = ' '
*     _SCOPE                      = '3'
*     _SYNCHRON                   = ' '
*     _COLLECT                    = ' '
            .
ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  STATUS_0100
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM STATUS_0100 .
  SET PF-STATUS 'G0100'.   " STATUS 설정 (펑션코드, 메뉴 등 설정) 
  SET TITLEBAR 'T0100'.    " 타이틀바 설정 
  IF GV_EDIT = 0.          " 조회모드인 경우 EDIT 버튼을 달아준다
    ST_SWITCH = VALUE #( TEXT      = 'Edit Mode'(S01)
                         ICON_ID   = '@3I@'
                         ICON_TEXT = 'Edit Mode'(S02)
                         QUICKINFO = 'Enables Edit Mode'(S03) ).
  ELSE.                    " 수정모드인 경우 READ 버튼을 달아준다
    ST_SWITCH = VALUE #( TEXT      = 'Read Mode'(S04)
                         ICON_ID   = '@10@'
                         ICON_TEXT = 'Read Mode'(S05)
                         QUICKINFO = 'Disables Edit Mode'(S06) ).
  ENDIF.
ENDFORM.