*&---------------------------------------------------------------------*
*&  Include           ZB10_PROGRAMTOP
*&---------------------------------------------------------------------*

* TYPE
TYPES: BEGIN OF TS_SFLIGHT.
       INCLUDE TYPE ZB10_SFLIGHT_TES.
       TYPES: LIGHT(1).
       TYPES: CELLTAB TYPE LVC_T_STYL.
TYPES: END OF TS_SFLIGHT.

* TABLES
TABLES: ZB10_SFLIGHT_TES.
*TABLES: smp_dyntxt, sscrfields.

* INTERNAL TABLE
DATA: GT_BUFFER  TYPE TABLE OF TS_SFLIGHT.
DATA: GT_SFLIGHT TYPE TABLE OF TS_SFLIGHT,
      GW_SFLIGHT LIKE LINE OF GT_SFLIGHT.

* DATA
DATA: OK_CODE    LIKE SY-UCOMM,   " 사용자 Action 저장
      GV_EDIT(4)Í TYPE I,          " 0 = 조회모드 1 = 수정모드
      GV_CNGE          ,          " 
      GV_READ_TABLE    ,          " 0 = 내가 lock을 잡은 경우 1 = foreign lock이 있어서 조회모드로 들어온 경우 
      ST_SWITCH TYPE SMP_DYNTXT.  " 조회<->수정 모드일 때 각각 동적으로 아이콘과 텍스트 100번 화면에서 변경해준다 

* MACRO
DEFINE CONFIRM .
  CALL FUNCTION 'POPUP_TO_CONFIRM'
    EXPORTING
     TITLEBAR                    = &1
     TEXT_QUESTION               = &2
     TEXT_BUTTON_1               = 'Yes'(P01)
     TEXT_BUTTON_2               = 'No'(P02)
     DISPLAY_CANCEL_BUTTON       = ABAP_TRUE
   IMPORTING
     ANSWER                      = &3.

END-OF-DEFINITION.

* SCREEN
SELECT-OPTIONS: SO_CAR FOR ZB10_SFLIGHT_TES-CARRID NO INTERVALS.
PARAMETERS: P_VAR TYPE DISVARIANT-VARIANT.