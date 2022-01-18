Webview2 <-> HTML(JSP) 동기적 구현
==========
CDHtmlDialog를 CDialog로 변환하여 Webview2 사용하기
------------

***

2022년 6월 15일 IE 지원 종료 예정에 따라 MFC 프로그램의 IE웹 사용을 모두 Webview2로 바꾸던 중,   
최초 Webview2의 생성 시에 Callback\<ICoreWebView2CreateCoreWebView2ControllerCompletedHandler\>의 핸들링 함수에서
Navigate 함수 호출 후 연달아 ExecuteScript 함수를 호출 할 경우 ExecuteScript 함수가 무시되는 상황이 발생했다.

또한 아래와 같이 OnInitDialog()에서 WebView를 Initialize한 뒤에 (m_Environment, m_Controller, m_WebView 모두 멤버변수)

    BOOL CDlg::OnInitDialog()
    {
      CDialog::OnInitDialog();
      InitializeWebView();
      return TRUE;
    }
    
    void CDlg::InitializeWebView()
    {
      HRESULT hr = CreateCoreWebView2EnvironmentWithOptions(nullptr, nullptr, nullptr,
		    Callback<ICoreWebView2CreateCoreWebView2EnvironmentCompletedHandler>(this, &CDlg::OnCreateEnvironmentCompleted).Get());
    }
    
    HRESULT CDlg::OnCreateEnvironmentCompleted(HRESULT result, ICoreWebView2Environment * environment)
    {
      HWND hWnd = GetSafeHwnd();
      m_Environment = environment;
      m_Environment->CreateCoreWebView2Controller(hWnd,
        Callback<ICoreWebView2CreateCoreWebView2ControllerCompletedHandler>(this, &CDlg::OnCreateCoreWebView2ControllerCompleted).Get());
      return S_OK;
    }
    
    HRESULT CDlg::OnCreateCoreWebView2ControllerCompleted(HRESULT result, ICoreWebView2Controller * controller)
    {
      if (result == S_OK)
      {
		    m_Controller = controller;
		    wil::com_ptr<ICoreWebView2> coreWebView2;
		    m_Controller->get_CoreWebView2(&coreWebView2);
		    coreWebView2.query_to<ICoreWebView2>(&m_WebView);
		    return S_OK;
	    }
	    else
	    {
		    return E_FAIL;
	    }
    }

아래와 같이 코드를 추가하여 매개변수를 Webview에 Navigate된 Html페이지에 넘겨주는 경우에도 마찬가지로 ExecuteScript가 무시되었다.

    BOOL CDlg::OnInitDialog()
    {
      CDialog::OnInitDialog();
      InitializeWebView();
      m_WebView->ExecuteScript("function('paramete'");
      return TRUE;
    }

***

이유는 Webview가 32비트 프로그램에서 비동기적으로 실행되기 때문이다.

따라서 Webview의 뷰가 모두 생성되기 전, 혹은 Navigate한 페이지의 DOM이 모두 생성되기 전에 ExecuteScript를 사용하면 그 호출은 무시된다.

***

동기적으로 시행하기 위한 방법이 여러가지 있겠지만, 나의 경우에는 Html 페이지가 모두 생성되면 window.chrome.webview.postMessage 함수를 통해 Dialog에 신호를 보내주고,
MFC에서 해당 신호를 받으면 매개변수를 넘겨주는 방식으로 해결했다.


Html Script 추가

    <script>
    window.onload = function () {
      window.chrome.webview.postMessage("DOMLoaded");
    }
    </script>

MFC Web Message Recieve 함수 추가

    HRESULT CDlg::OnCreateCoreWebView2ControllerCompleted(HRESULT result, ICoreWebView2Controller * controller)
    {
      if (result == S_OK)
      {
		    m_Controller = controller;
		    wil::com_ptr<ICoreWebView2> coreWebView2;
		    m_Controller->get_CoreWebView2(&coreWebView2);
		    coreWebView2.query_to<ICoreWebView2>(&m_WebView);
        
        
		    EventRegistrationToken token;
		    m_WebView->add_WebMessageReceived(Callback<ICoreWebView2WebMessageReceivedEventHandler>(this, &CDlg::WebMessageReceived).Get(), &token);
        
		    return S_OK;
	    }
	    else
	    {
		    return E_FAIL;
	    }
    }
    
    HRESULT CSmsPreviewMmsDlg::WebMessageReceived(ICoreWebView2 * sender, ICoreWebView2WebMessageReceivedEventArgs * args)
    {
	    LPWSTR pwStr;
	    args->TryGetWebMessageAsString(&pwStr);
	    CString rm = CW2A(pwStr);
	    if (rm == "DOMLoaded") {
        m_WebView->ExecuteScript(L"pageInit('parameter')", Callback<ICoreWebView2ExecuteScriptCompletedHandler>(
          [this](HRESULT error, PCWSTR result) -> HRESULT
          {
            if (error != S_OK)
            {
              TRACE("ExecuteScript failed");
            }
            return S_OK;
          }).Get());
      }
      return S_OK;
    }
    
Html에서 DOM 로드가 끝나면 "DOMLoaded"라는 신호를 MFC로 보내주고, MFC에서는 해당 신호를 받으면 Html의 함수를 매개변수와 함께 실행하도록하는 코드이다.

***

참고 사이트   
https://github.com/MicrosoftEdge/WebView2Feedback/issues/838   
https://github.com/MicrosoftEdge/WebView2Feedback/issues/840
