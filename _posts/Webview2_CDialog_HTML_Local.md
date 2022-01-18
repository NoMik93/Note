CDialog-WebView2에서 로컬의 HTML파일에 접근하기
==========
경로를 이용하여 로컬의 HTML파일을 Navigate하기
------------

***

프로그램에 내장된 Html Resource를 사용해서 Webview2를 Navigate하면, 브라우저 보안정책으로 인해 ActiveX를 사용하지 않고는 해당 Html에서 로컬 파일에 접근할 수 없다.
따라서 로컬 파일에 접근하기 위해서는 로컬 Html 파일을 사용하고 해당 파일에서 로컬 파일에 접근하도록 하면 된다.

***
프로그램 파일(.exe)과 Html파일이 같은 경로에 있다고 가정했을 때 아래와 같이 로컬 Html파일을 Navigate 할 수 있다.

    TCHAR path[FILENAME_MAX];
		GetModuleFileName(NULL, path, FILENAME_MAX);
		CString s = path;
		CString szResult = s.Left(s.ReverseFind('\\') + 1);
		HRESULT hr = m_WebView->Navigate(CA2W(szResult + "Html파일명.htm")); //m_WebView는 멤버 변수로 이미 생성된 상태
		if (hr != S_OK) {
			TRACE("Failed to Navigate webview\n");
		}
