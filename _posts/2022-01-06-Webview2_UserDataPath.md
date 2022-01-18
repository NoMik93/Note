---
layout: post
title: WebView2에서 UserDataPath 설정하기
subtitle: WebView2 사용시 생성되는 EBWebView 폴더의 경로 설정하기
gh-repo: NoMik93/Note
published: true
tags:
  - WebView2
---

***

MFC에서만 그런지는 모르겠으나, WebView2에서 특정 폴더에 디렉토리 및 파일을 생성 할 수 없다.

"C:\Program Files" 와 "C:\Program Files (x86)" 같은 프로그램 파일 폴더도 그렇다.
따라서 해당 폴더에 설치된, 혹은 복사된 프로그램에서 UserDataPath를 설정하지 않고 WebView2를 사용하면
"microsoft edge 는 데이터 디렉터리를 읽고 쓸 수 없습니다. (microsoft edge cannot read and write to its data directory.)"라는 경고가 뜨게 된다.

왜냐하면 WebView2에서 User Data가 저장되는 기본 경로는 "(해당 프로그램의 위치)\(해당프로그램이름.WebView2)\EBWebView"이기 때문이다.
따라서 이를 해결하기 위해서는 WebView2의 EBWebView가 생성되는 UserDataPath를 설정해주면 된다.

***

아래와 같이 WebView2 환경을 설정하며 생성할 때, 두 번째 Parameter(userDataPath, PCWSTR타입)로 경로를 명시해주면 된다.

    CreateCoreWebView2EnvironmentWithOptions(nullptr, userDataPath, nullptr,
		  Callback<ICoreWebView2CreateCoreWebView2EnvironmentCompletedHandler>(this,
        &CDialog::OnCreateEnvironmentCompleted).Get());//CDialog는 해당 다이얼로그명, OnCreateEnvironmentCompleted는 Controller 및 WebView 생성 함수로 작성

이때 해당 **userDataPath** 부분은 예를 들면 아래와 같이 %APPDATA% 폴더의 프로그램명 폴더로 설정할 수도 있다.

    CString userDataPath = getenv("APPDATA"); //%APPDATA% 경로 가져오기
	  userDataPath += "\\(프로그램명)"; //경로 끝에 프로그램명 폴더 붙이기
	  if (!PathFileExistsA(userDataPath)) { //해당 폴더가 없으면
		  CreateDirectory(userDataPath, NULL); //폴더 생성
	  }
