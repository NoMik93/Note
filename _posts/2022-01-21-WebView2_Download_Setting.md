---
published: true
layout: post
title: Webview2 다운로드 시, '다른 이름으로 저장' 띄우기
subtitle: Webview2 DownloadStarting Event 추가하기
gh-repo: NoMik93/Note
tags:
  - WebView2
---


***

MFC 프로그램에서 WebView2로 웹페이지를 띄우고 그 웹페이지에서 파일을 저장하려고 하면, IE에서의 다운로드와 크로미움 기반 웹에서의 다운로드를 따로 지정해야 한다. 
(물론 22년 6월이면 IE가 지원 종료되긴 하지만, 아직까지는...)   
그리고 BLOB을 다운로드 하는 과정에서 아래와 같이 HTML파일의 코드를 짤 수 있는데, IE에서야 자동으로 '다른 이름으로 저장' 다이얼로그를 띄워서 원하는 위치로 저장하게 해주지만, 
크로미움 기반 웹에서는 저장 위치를 설장하는 창이 안 뜨고 바로 download 폴더에 다운로드 된다.


    function downloadURI(blob, name) {
        if (window.navigator && window.navigator.msSaveOrOpenBlob) {
            // IE에서 동작
            window.navigator.msSaveBlob(blob, name);
        } else {
            // 크롬에서 동작
            var link = document.createElement('a');
            link.download = name;
            link.href = URL.createObjectURL(blob);
            link.click();
            delete link;
        }
    }

a tag를 사용하지 않고 저장 위치와 파일 이름을 지정할 수 있는 다른방법을 찾아봤으나, HTML에서는 찾지 못했다. 
따라서 이를 해결하기 위해 MFC에서 '다른 이름으로 저장' 다이얼로그를 띄우고 그 위치로 저장하도록 했다.

***

MFC에서 WebView2에 다운로드 이벤트를 설정하는 코드는 아래와 같다. 이 때 downloadPath라는 CString 형태의 전역변수에서 저장할 Path를 가져오는데, 
이 변수의 value가 바뀐 뒤에 동기적으로 다운로드가 진행되면 해당 위치로 저장이 된다.

    HRESULT 다이얼로그명::OnCreateCoreWebView2ControllerCompleted(HRESULT result, ICoreWebView2Controller * controller)
    {
      if (result == S_OK)
      {
          m_Controller = controller;
          wil::com_ptr<ICoreWebView2> coreWebView2;
		      wil::com_ptr<ICoreWebView2_4> coreWebView2_4;
          m_Controller->get_CoreWebView2(&coreWebView2);
          coreWebView2_4 = coreWebView2.query<ICoreWebView2_4>(); //add_DOMContentLoaded 함수는 ICoreWebView2_4에 존재하므로 해당 Interface로 Query
          EventRegistrationToken downloadtoken;
          coreWebView2_4->add_DownloadStarting(Callback<ICoreWebView2DownloadStartingEventHandler>(  //다운로드 시작 이벤트
			      [this](
              ICoreWebView2* sender, ICoreWebView2DownloadStartingEventArgs* args)->HRESULT {
            args->put_ResultFilePath(CA2W(downloadPath));//downloadPath는 다른 함수에서 '다른 이름으로 저장' 다이얼로그를 통해 값을 변경 할 수 있도록 전역변수로 사용.
            return S_OK;
		      }).Get(), &downloadtoken);
          
          m_WebView = coreWebView2_4.query<ICoreWebView2>(); //기본 기능을 사용하기 위해 ICoreWebView2 Interface로 Query
          HRESULT hresult = m_WebView->Navigate("사이트 주소"));
      }
      else
      {
          TRACE("Failed to create webview\n");
          return E_FAIL;
      }
      return S_OK;
    }

위에서 볼 수 있듯이 add_DownloadStarting 함수를 이용해서 이벤트를 추가할 수 있다. 
이때 add_DownloadStarting 함수는 ICoreWebView2_4 Interface에 정의되어 있기 때문에, 해당 Interface로 Query하여 함수를 사용한다. 
그리고 Navigate와 같은 함수는 ICoreWebView2에 정의되어 있기에 다시 ICoreWebView2로 Query하면 해당 함수들을 사용할 수 있다.

그리고 MFC에서 '다른 이름으로 저장' 다이얼로그를 띄우기 위해 먼저 HTML에서 다운로드 버튼 클릭 이벤트를 추가한다. 
나는 JSON 형태로 MFC에 Parameter를 넘겨줄건데, 이 때 evnet key를 사용하여 이벤트를 구분했다.

    $( this ).find( '.btn_down' ).click( function( e ) {
      e.preventDefault();
      var obj = new Object();
      obj.event = "DownloadPrepare";
      obj.fileName = "파일이름";
      window.chrome.webview.postMessage(JSON.stringify(obj));
    }

MFC에서 위의 Message를 받는 함수는 아래와 같은데, Web Message Recieve 이벤트를 추가하는 법은 '[Webview2-HTML(JSP) 동기적 구현](https://nomik93.github.io/Note/2021-12-28-Webview2_Synchronous/)' 포스트에서 볼 수 있다.

    HRESULT CRecordListDlg::WebMessageReceived(ICoreWebView2 * sender, ICoreWebView2WebMessageReceivedEventArgs * args)
    {
      LPWSTR pwStr;
      args->TryGetWebMessageAsString(&pwStr);
      std::string receivedMessage = CW2A(pwStr);
      if (!receivedMessage.empty())
      {
        Json::Reader jReader;
        Json::Value jValue;
        if (jReader.parse(receivedMessage, jValue)) {
          CString eventMessage = jValue["event"].asCString();
          if (eventMessage == "DownloadPrepare") { //이벤트 이름이 DownloadPrepare면,
            CString fileName = jValue["fileName"].asCString();
            std::thread getPathThread(GetDownloadPath, fileName); //Thread에서 '다른 이름으로 저장' 다이얼로그을 띄우는 함수 호출
            getPathThread.join(); //스레드 종료까지 대기
            if (downloadPath != "") { //downloadPath는 전역변수
              Json::Value root(Json::objectValue);
              root["fileName"] = (LPSTR)(LPCSTR)fileName; //크로미움 기반 웹에서는 이미 파일 다운로드 위치와 파일 이름이 DownloadStarting 이벤트로 정해져서 의미가 없으나,
              //IE 기반에서 사용하기 위해 파일 이름을 JSON에 추가해서 넘겨줌
              
              CString exe = "download('";
              exe += root.toStyledString().c_str();
              exe += "')";
              
              HRESULT hr = m_WebView->ExecuteScript(CA2W(DeleteNewLine(exe)), Callback<ICoreWebView2ExecuteScriptCompletedHandler>(
                [this](HRESULT error, PCWSTR result) -> HRESULT //HTML의 download script를 호출하여 다운로드 명령.
              {
                if (error != S_OK) {\
                  return E_FAIL;
                }
                return S_OK;
              }).Get());
              
              if (hr != S_OK) {
                AfxMessageBox("파일 다운로드에 실패하였습니다.");
                return E_FAIL;
              }
            }
            return S_OK;
          }
        }
      }
      return E_FAIL;
    }

여기서 굳이 GetDownloadPath라는 함수를 만들고 Thread에서 실행시킨 이유는, WebView2는 단일 Thread로 진행되기 때문에 위의 이벤트 도중 다른 다이얼로그를 위에 띄워버리면
'Running a message loop synchronously in an event handler in Webview can cause reentrancy issue.'라는 에러가 뜬다.
(바로 뜨는 건 아니고, 내 경우에는 다른 다이얼로그를 WebView2 위에 20초쯤 띄워두면 WebView2의 Thread가 종료되고 실행되길 반복하다가 어느 순간 변수 포인터를 못가져오면서 해당 에러가 뜨게된다.)

따라서 Thread를 사용했고 async나 future, promise같은 함수를 사용해도 되지만 하나의 Thread만 사용할 거라서 그냥 Thread를 사용했다.

그리고 해당 Thread에서 호출하는 GetDownloadPath 함수는 아래와 같다.

    void CRecordListDlg::GetDownloadPath(CString fileName)
    {
      TCHAR document_path[MAX_PATH] = "";
      SHGetSpecialFolderPath(NULL, document_path, CSIDL_MYDOCUMENTS, 0);
      char szFilter[] = "MP3 File (*.mp3, *.MP3)"; //나의 경우 mp3 파일이기에 mp3로 필터 적용
      CFileDialog dlg(FALSE, "mp3", fileName, OFN_OVERWRITEPROMPT, szFilter);
      dlg.m_ofn.lpstrInitialDir = document_path;
      if (IDOK == dlg.DoModal()) { //파일 경로를 받아오면 downloadPath 변수에 저장.
        downloadPath = dlg.GetPathName();
      }
      else {
        downloadPath = "";
      }
    }

***

참고 사이트   
[https://docs.microsoft.com/en-us/microsoft-edge/webview2/reference/win32/icorewebview2_4?view=webview2-1.0.1054.31#add_downloadstarting](https://docs.microsoft.com/en-us/microsoft-edge/webview2/reference/win32/icorewebview2_4?view=webview2-1.0.1054.31#add_downloadstarting)   
[https://docs.microsoft.com/en-us/microsoft-edge/webview2/concepts/threading-model#re-entrancy](https://docs.microsoft.com/en-us/microsoft-edge/webview2/concepts/threading-model#re-entrancy)
