---
layout: post
title: CDialog-WebView2에서 MFC의 Resource에 포함된 HTML파일에 접근하기
subtitle: CDHtmlDialog 사용하지 않고 Webview2에서 프로젝트에 포함된 HTML Resource 사용하기
gh-repo: NoMik93/Note
published: true
tags:
  - WebView2
---

***

프로그램의 런타임 라이브러리를 Webview2로 변경하기 위해서 CDHtmlDialog를 CDialog로 바꾸면, MFC에서 자동으로 연결해주던 Html Resource를 사용할 수 없게 된다.   
따라서 CDialog에서 Html Resource에 접근하기 위해서는 Afx Handle에서 직접 해당 Resource를 검색해야한다.

***

    HINSTANCE hInstance = AfxGetResourceHandle();
		CString strResourceURL;
		LPTSTR lpszModule = new TCHAR[_MAX_PATH];

		if (GetModuleFileName(hInstance, lpszModule, _MAX_PATH)) //현재 실행 경로를 가져올 수 있으면
		{
			HRSRC hResource = FindResource(hInstance, MAKEINTRESOURCE(IDR_HTML), RT_HTML); //IDR_HTML에 해당하는 부분은 프로젝트명.rc 파일의 html에서 찾을 수 있다.
			HGLOBAL  hMemory = LoadResource(hInstance, hResource);
			std::size_t size_bytes = SizeofResource(hInstance, hResource);
			void* ptr = LockResource(hMemory);
			CString html = static_cast<LPCTSTR>(ptr);
			m_WebView->NavigateToString(CA2W(html)); //m_WebView는 멤버 변수로 이미 생성되어 있어야 한다.
		}
		else {
			TRACE("Failed to Navigate webview\n");
		}

***

다만 위의 방식으로 프로그램에 포함된 Html Resource에 접근하여 브라우저를 Navigate했을 때, 만약 해당 Html에서 로컬 파일에 접근하려하면 'Not allowed to load local resource' 에러가 발생한다.   
즉 해당 Html에서 로컬 자원에 접근할 수 없다. 왜냐하면 서버의 특정 파일에 접근 할 수 있다면 보안 이슈가 발생할 수 있기 때문에 대부분의 브라우저에서 로컬 자원의 접근을 막아두기 때문이다.   
ActiveX를 사용하면 로컬 파일에 접근할 수 있다고 하는데, 나는 그냥 위의 방식으로 프로젝트의 내장된 Html Resource에 접근하는 것을 포기했다.   
대신에 프로그램 설치파일에 Html 파일을 포함시켜서 로컬 Html에서 로컬 Resource에 접근하도록 했다.

***

참고 사이트   
https://github.com/MicrosoftEdge/WebView2Feedback/issues/772
