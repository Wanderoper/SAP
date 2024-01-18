*&---------------------------------------------------------------------*
*&  Include           ZB10_PROGRAMPAI
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Module  EXIT_BACK  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE EXIT_BACK INPUT.
  CASE OK_CODE.
    WHEN 'BACK'.
      PERFORM FREE_CONTROL_RESOURCES.
      LEAVE TO SCREEN 0.
  ENDCASE.

ENDMODULE.
*&---------------------------------------------------------------------*
*&      Module  USER_COMMAND_0100  INPUT
*&---------------------------------------------------------------------*
*       유저 액션을 처리해준다
*----------------------------------------------------------------------*
MODULE USER_COMMAND_0100 INPUT.
  CASE OK_CODE.
    WHEN 'BACK'.                       " 이전 화면으로 나가기
      PERFORM FREE_CONTROL_RESOURCES.  " 나가기 전 alv object, lock release 해주기
      LEAVE TO SCREEN 0.               " 0번 (이전화면 있으면 이전화면, 없으면 프로그램 종료) 
    WHEN 'EXIT'.                       " 프로그램 종료
      PERFORM FREE_CONTROL_RESOURCES.  " 동일
      LEAVE PROGRAM.                   " 이전화면이 아닌 프로그램 종료
    WHEN 'CANC'.                       " 입력 내용 취소  
*      PERFORM FREE_CONTROL_RESOURCES. " 새로고침으로 바꾸기
*      LEAVE TO SCREEN 0.
    WHEN 'SWITCH'.                     " 100번 화면에 동적으로 변경하기 위한 버튼 클릭 (조회<->수정모드 변경)
      IF GV_READ_TABLE = ABAP_TRUE.    " gv_read_table : 사용자가 조회모드로(foreign lock) 들어온 경우 수정버튼 눌러도 필드가 열리면 안됨
        MESSAGE E011. " READ-ONLY mode. 
      ELSE.
        PERFORM SWITCH_EDIT_MODE.      " foreign lock이 걸려있어서 조회모드로 들어온 경우가 아니라면 수정이 가능하므로 변경해줌
      ENDIF.
  ENDCASE.


ENDMODULE.