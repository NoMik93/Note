---
published: true
layout: post
title: Webview2-HTML(JSP) 동기적 구현2
subtitle: Webview2 DOMContentLoaded Event 추가하기
gh-repo: NoMik93/Note
tags:
  - WebView2
---


***

이전에 올렸던 [Webview2-HTML(JSP) 동기적 구현](https://nomik93.github.io/Note/2021-12-28-Webview2_Synchronous) 포스트에서 사용했던 방식보다 훨씬 나은 방식을 알게 되어서 다시 작성한다.

CHtmlDialog의 DOMContentLoaded 이벤트를 추가하는 것과 마찬가지 방식이 WebView2에도 존재하는데, 그것은 아래와 같이 구현할 수 있다.

    HRESULT 다이얼로그명::OnCreateCoreWebView2ControllerCompleted(HRESULT result, ICoreWebView2Controller * controller)
    {
      if (result == S_OK)
      {
          m_Controller = controller;
          wil::com_ptr<ICoreWebView2> coreWebView2;
          wil::com_ptr<ICoreWebView2_2> coreWebView2_2;
          m_Controller->get_CoreWebView2(&coreWebView2);
          coreWebView2_2 = coreWebView2.query<ICoreWebView2_2>(); //add_DOMContentLoaded 함수는 ICoreWebView2_2에 존재하므로 해당 Interface로 Query
          EventRegistrationToken m_DOMContentLoadedToken;
          coreWebView2_2->add_DOMContentLoaded(Callback<ICoreWebView2DOMContentLoadedEventHandler>(
              [this](ICoreWebView2* sender, ICoreWebView2DOMContentLoadedEventArgs* args)->HRESULT {
              m_WebView->ExecuteScript(L"ReceiveMessage('*v*')", nullptr); //DOMLoad가 완료되면 시행될 부분
              return S_OK;
          }).Get(), &m_DOMContentLoadedToken);
          
          coreWebView2_2.query_to<ICoreWebView2>(&m_WebView);
          HRESULT hresult = m_WebView->Navigate("사이트 주소"));
      }
      else
      {
          TRACE("Failed to create webview\n");
          return E_FAIL;
      }
      return S_OK;
    }

위에서 볼 수 있듯이 add_DOMContentLoaded 함수를 이용해서 이벤트를 추가할 수 있다. 이때 add_DOMContentLoaded 함수는 ICoreWebView2_2 Interface에 정의되어 있기 때문에, 해당 Interface로 Query하여 함수를 사용한다. 그리고 Navigate와 같은 함수는 ICoreWebView2에 정의되어 있기에 다시 ICoreWebView2로 Query하면 해당 함수들을 사용할 수 있다.   
또한 Query는 "(Query받을변수) = (Query대상변수).query\<인터페이스명\>()"으로 해도 되고, "(Query대상변수).query_to\<인터페이스명\>(&Query받을변수)"를 사용해도 된다.

***

참고 사이트   
[https://docs.microsoft.com/en-us/microsoft-edge/webview2/reference/win32/icorewebview2_2?view=webview2-1.0.1054.31#add_domcontentloaded](https://docs.microsoft.com/en-us/microsoft-edge/webview2/reference/win32/icorewebview2_2?view=webview2-1.0.1054.31#add_domcontentloaded)
