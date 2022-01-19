---
layout: post
title: 프로젝트의 지원 운영체제 최소 버전이 낮다는 오류
subtitle: 'WINVER, _WIN32_WINNT, _WIN32_WINDOWS, _WIN32_IE'
gh-repo: NoMik93/Note
published: true
tags:
  - WINVER
  - _WIN32_WINNT
  - _WIN32_WINDOWS
  - _WIN32_IE
---

***
MFC에 WebView2를 적용하기 위해 wrl.h 헤더를 include 시켰을 때,
아래와 같은 오류가 발생했다.

    오류 코드 E0035   
    #error 지시문: NTDDI_VERSION to be #defined at least to NTDDI_VISTA or greater. Please change the definition of NTDDI_VERSION in your project properties or precompiled header.   
    #error 지시문: MFC does not support WINVER less than 0x0501. Please change the definition of WINVER in your project properties or precompiled header.

afv_w32.h 파일에 명시된 에러로, 아래의 방식으로 해결했다.

***

해당 프로젝트의 stdafx.h 파일에서 수정 가능하다.
파일 최상위의 부분인

    #ifndef WINVER				// Windows 95 및 Windows NT 4 이후 버전에서만 기능을 사용할 수 있습니다.
    #define WINVER  0x0400	//0x0400		// Windows 98과 Windows 2000 이후 버전에 맞도록 적합한 값으로 변경해 주십시오.
    #endif

    #ifndef _WIN32_WINNT		// Windows NT 4 이후 버전에서만 기능을 사용할 수 있습니다.
    #define _WIN32_WINNT  0x0400	//0x0400		// Windows 98과 Windows 2000 이후 버전에 맞도록 적합한 값으로 변경해 주십시오.
    #endif						

    #ifndef _WIN32_WINDOWS		// Windows 98 이후 버전에서만 기능을 사용할 수 있습니다.
    #define _WIN32_WINDOWS  0x0400 // Windows Me 이후 버전에 맞도록 적합한 값으로 변경해 주십시오.
    #endif

    #ifndef _WIN32_IE			// IE 4.0 이후 버전에서만 기능을 사용할 수 있습니다.
    #define _WIN32_IE 0x0400	//0x0400	// IE 5.0 이후 버전에 맞도록 적합한 값으로 변경해 주십시오.
    #endif

부분을 필요한 최소 버전에 맞게

    #ifndef WINVER				// Windows VISTA 이후 버전에서만 기능을 사용할 수 있습니다.
    #define WINVER  0x0600	//0x0600		// Windows 98과 Windows 2000 이후 버전에 맞도록 적합한 값으로 변경해 주십시오.
    #endif

    #ifndef _WIN32_WINNT		// Windows VISTA 이후 버전에서만 기능을 사용할 수 있습니다.
    #define _WIN32_WINNT  0x0600	//0x0600		// Windows 98과 Windows 2000 이후 버전에 맞도록 적합한 값으로 변경해 주십시오.
    #endif						

    #ifndef _WIN32_WINDOWS		// Windows VISTA 이후 버전에서만 기능을 사용할 수 있습니다.
    #define _WIN32_WINDOWS  0x0600 // Windows Me 이후 버전에 맞도록 적합한 값으로 변경해 주십시오.
    #endif

    #ifndef _WIN32_IE			// IE 9.0 이후 버전에서만 기능을 사용할 수 있습니다.
    #define _WIN32_IE 0x0900	//0x0900	// IE 5.0 이후 버전에 맞도록 적합한 값으로 변경해 주십시오.
    #endif

위와 같은 형식으로 수정하면 된다.

***

또한 버전에 대한 define 값은 sdkddkver.h 파일에서 확인 가능하다.

    //
    // _WIN32_WINNT version constants
    //
    #define _WIN32_WINNT_NT4                    0x0400
    #define _WIN32_WINNT_WIN2K                  0x0500 
    #define _WIN32_WINNT_WINXP                  0x0501
    #define _WIN32_WINNT_WS03                   0x0502
    #define _WIN32_WINNT_WIN6                   0x0600
    #define _WIN32_WINNT_VISTA                  0x0600
    #define _WIN32_WINNT_WS08                   0x0600
    #define _WIN32_WINNT_LONGHORN               0x0600
    #define _WIN32_WINNT_WIN7                   0x0601
    #define _WIN32_WINNT_WIN8                   0x0602
    #define _WIN32_WINNT_WINBLUE                0x0603
    #define _WIN32_WINNT_WINTHRESHOLD           0x0A00 /* ABRACADABRA_THRESHOLD*/
    #define _WIN32_WINNT_WIN10                  0x0A00 /* ABRACADABRA_THRESHOLD*/

    //
    // _WIN32_IE_ version constants
    //
    #define _WIN32_IE_IE20                      0x0200
    #define _WIN32_IE_IE30                      0x0300
    #define _WIN32_IE_IE302                     0x0302
    #define _WIN32_IE_IE40                      0x0400
    #define _WIN32_IE_IE401                     0x0401
    #define _WIN32_IE_IE50                      0x0500
    #define _WIN32_IE_IE501                     0x0501
    #define _WIN32_IE_IE55                      0x0550
    #define _WIN32_IE_IE60                      0x0600
    #define _WIN32_IE_IE60SP1                   0x0601
    #define _WIN32_IE_IE60SP2                   0x0603
    #define _WIN32_IE_IE70                      0x0700
    #define _WIN32_IE_IE80                      0x0800
    #define _WIN32_IE_IE90                      0x0900
    #define _WIN32_IE_IE100                     0x0A00
    #define _WIN32_IE_IE110                     0x0A00  /* ABRACADABRA_THRESHOLD */
    
***

참고사이트   
[https://stackoverflow.com/questions/14643962/error-c1189-error-this-file-requires-win32-winnt-to-be-defined-at-least-to](https://stackoverflow.com/questions/14643962/error-c1189-error-this-file-requires-win32-winnt-to-be-defined-at-least-to)
