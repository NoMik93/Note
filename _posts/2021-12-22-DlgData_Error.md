---
layout: post
title: 다이얼로그에서 해당 컨트롤을 찾을 수 없다는 오류
subtitle: 'Debug Assertion Failed!   
File: ~stlmfc\src\mfc\dlgdata.cpp   
Line: 40'
gh-repo: NoMik93/Note
published: true
tags:
  - dlgdata.cpp
---
***

해당 내용에 대해서 검색했을 때 컨트롤을 지우고 다이얼로그 cpp 파일에서 IDX_Control 선언을 지우지 않아서 생기는 오류라고 나왔으나,   
반대로 IDX_Control 선언이 있지만 '프로젝트명.rc' 파일에서 컨트롤을 찾을 수 없을 경우에도 해당되는 에러이다.

즉, 사용 중인 컨트롤이라면 다이얼로그의 cpp 파일에서 아래와 같이 선언되어있을 때,
    
    void 다이얼로그명::DoDataExchange(CDataExchange* pDX)
    {
      상속받은다이얼로그명::DoDataExchange(pDX);
      DDX_Control(pDX, IDC_매크로명, 변수명);
    }

'IDC_매크로명'에 해당하는 컨트롤이 리소스 파일 폴더(필터) 내부의 "프로젝트명.rc" 파일에 없을 때도 나올 수 있다.

따라서 IDX_Control과 .rc파일(리소스 관리 파일)의 링크가 정상적이지 않을 때 발생하는 에러이다.

***

예를 들어 Test프로젝트에서 CTestDlg.cpp 파일에 아래와 같이 DoDataExchange가 선언되어 있다고 하자.

    void CTestDlg::DoDataExchange(CDataExchange* pDX)
    {
      CDialog::DoDataExchange(pDX);
      DDX_Control(pDX, IDC_editText, m_ctlText);
      DDX_Control(pDX, IDC_btn_OK, m_ctlOK);
      DDX_Control(pDX, IDC_btn_Cancel, m_ctlCancel);
      DDX_Control(pDX, IDC_lst_FileList, m_ctlFileList);
    }

이때 리소스 파일의 Test.rc 파일을 우클릭하여 코드 보기를 했을 때,

    IDD_TEST DIALOG 0, 0, 300, 500
    STYLE DS_SETFONT | DS_FIXEDSYS | WS_CHILD | WS_SYSMENU
    FONT 8, "MS Shell Dlg", 400, 0, 0x1
    DEGIN
      EDITTEXT  IDC_editText, 5, 24, 280, 52, ES_MULTILINE | ES_AUTOVSCROLL | ES_WANTRETURN | NOT WS_BORDER, WS_EX_TRANSPARENT
      CONTROL "OK", IDC_btn_OK, "Button", BS_OWNERDRAW, 15, 80, 100, 24
      CONTROL "", IDC_lst_FileList, "SysListView32",LVS_REPORT | LVS_ALIGNLEFT | LVS_NOCOLUMNHEADER, 10, 120, 280, 240
    END

이와 같이 정의되어 있다면
IDC_btn_Cancel에 대해 컨트롤이 없기 때문에 에러가 나는 것이다.
