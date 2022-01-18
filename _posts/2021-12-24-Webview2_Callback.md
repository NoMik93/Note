---
layout: post
title: CDialog에서 WRL의 Callback 식별자 사용하기
subtitle: WebView2 사용시 필요한 Callback<ICoreWebView2ExecuteScriptCompletedHandler> 식별자 사용하기
gh-repo: NoMik/Note
published: true
tags:
  - WebView2
---

***

WebView2 사용시 Environment와 Controller, Webview 생성을 위해 Callback이란 식별자를 사용하는데,   minwindef.h에서 __stdcall으로 정의된 CALLBACK이 아니라 wrl.h의 Callback을 호출해야한다. 

#include <wrl.h>를 선언하더라도 '식별자 "Callback"이 정의되어 있지 않습니다.' 라는 에러가 뜨게 되면, 아래와 같이 namespace를 사용하여 해결 할 수 있다.

    using namespace Microsoft::WRL;   
