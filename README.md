# PowerShell Invoke-WebRequest プロキシ Guide

[![Promo](https://github.com/bright-kr/LinkedIn-Scraper/blob/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/proxy-types) 

이 **Invoke-WebRequest PowerShell プロキシ 가이드**를 끝까지 읽으면 다음을 알게 됩니다:

1. [PowerShell Invoke-WebRequest란 무엇입니까?](#what-is-powershell-invoke-webrequest)
2. [Invoke-WebRequest 설치](#installing-invoke-webrequest)  
   2.1. [Windows](#windows)  
   2.2. [macOS and Linux](#macos-and-linux)
3. [PowerShell에서 プロキシ를 시작하기 위한 사전 요구 사항](#prerequisites-to-get-started-with-a-proxy-in-powershell)
4. [Invoke-WebRequest에서 HTTP プロキシ를 지정하는 방법](#how-to-specify-an-http-proxy-in-invoke-webrequest)  
   4.1. [コマンドライン オプション 사용](#using-a-command-line-option)  
   4.2. [환경 변수를 사용](#using-environment-variables)
5. [PowerShell에서 HTTPS 및 SOCKS プロキシ를 사용하는 방법](#how-to-use-https-and-socks-proxies-in-powershell)
6. [알아야 할 팁과 트릭](#tips-and-tricks-you-need-to-know)  
   6.1. [PowerShell プロキシ 구성 무시](#ignore-the-powershell-proxy-configuration)  
   6.2. [SSL 인증서 오류 방지](#avoid-ssl-certificate-errors)
7. [어떤 PowerShell プロキシ를 사용해야 합니까?](#which-powershell-proxy-should-you-use)

이제 시작해 보겠습니다!

---

## What Is PowerShell Invoke-WebRequest?

**Invoke-WebRequest**는 웹 서버 및 웹 서비스로 **HTTP, HTTPS, FTP** リクエスト를 보내기 위한 PowerShell cmdlet입니다. 기본적으로 서버가 생성한 レスポンス를 자동으로 파싱하고, 폼, 링크, 이미지 또는 기타 중요한 HTML 요소 컬렉션을 반환합니다.

일반적으로 REST API에 액세스하거나, 웹에서 파일을 다운로드하거나, 웹 서비스와 상호작용하는 데 사용됩니다. 아래는 **Invoke-WebRequest** リクエスト의 기본 구문입니다:

```powershell
Invoke-WebRequest [-Uri] <Uri> [-Method <WebRequestMethod>] [-Headers <IDictionary>] [-Body <Object>]
```

**기억해야 할 주요 パラメータ**:

- Uri: リクエスト가 전송되는 웹 리소스의 URI입니다.
- Method: リクエスト에 사용할 HTTP 메서드입니다(예: GET, POST, PUT, DELETE).
- Invoke-WebRequest는 기본적으로 GET リクエスト를 전송합니다.
- Headers: リクエスト에 포함할 추가 HTTP ヘッダー입니다.
- Body: 서버로 전송할 リクエスト 본문입니다。
 
보시는 것처럼 필수 인수는 <Uri>뿐입니다. 따라서 간단히 말해, 지정된 URI에 GET リクエスト를 수행하는 가장 단순한 구문은 다음과 같습니다:

```powershell
Invoke-WebRequest <Uri>
```

## Installing Invoke-WebRequest

Invoke-WebRequest를 사용하려면 PowerShell이 필요합니다. PowerShell을 설치하고 Invoke-WebRequest cmdlet에 접근하는 방법을 알아보겠습니다!

### Windows

먼저 Windows PowerShell과 PowerShell은 서로 다른 것임을 이해해야 합니다. Windows PowerShell은 Windows에 포함되어 제공되는 PowerShell 버전(최신 버전 5.1)입니다. 여기에는 Invoke-WebRequest cmdlet이 포함됩니다. 최신 Windows 릴리스를 사용 중이라면 바로 사용 가능합니다! 구형 버전의 경우, 공식 PowerShell 설치 가이드를 따르십시오.

Invoke-WebRequest의 일부 기능은 PowerShell 7.x부터만 제공됩니다. 설치 방법에 대한 자세한 내용은 Windows PowerShell 5.1에서 PowerShell 7로의 공식 마이그레이션 가이드를 따르십시오. PowerShell 7.x는 새 디렉터리에 설치되며 Windows PowerShell 5.1과 나란히(side-by-side) 실행됩니다.

Windows 머신에서 현재 PowerShell 버전을 확인하려면 다음을 실행할 수 있습니다:

```powershell
$PSVersionTable
```

PowerShell 7.x에서는 다음과 비슷한 출력이 표시될 수 있습니다:

```powershell
PSVersion                   7.4.1
PSEdition                   Core
GitCommitId                 7.4.1
OS                          Microsoft Windows 10.0.22631
Platform                    Win32NT
PSCompatibleVersions        {1.0, 2.0, 3.0, 4.0…}
PSRemotingProtocolVersion   2.3
SerializationVersion        1.1.0.1
WSManStackVersion           3.0
```

### macOS and Linux

PowerShell 7.x는 macOS와 Linux 모두에 설치할 수 있습니다. 그러나 Invoke-WebRequest cmdlet에 접근하기 위해서만 이 OS들에 PowerShell 생태계 전체를 설치하는 것은 큰 의미가 없습니다. 대신 macOS 및 대부분의 Linux 배포판에 기본으로 설치되어 있고 동일한 기능을 제공하는 curl을 사용할 수 있습니다. 자세한 내용은 [curl プロキシ 가이드](https://brightdata.co.kr/blog/proxy-101/curl-with-proxies)에서 확인하십시오.

## Prerequisites to Get Started with a Proxy in PowerShell

プロキシ는 클라이언트와 대상 서버 사이에서 중개자 역할을 합니다. 즉, リクエスト를 가로채 서버로 전달하고, レスポンス를 받은 뒤 다시 사용자에게 전송합니다. 이 방식으로 대상 서버는 リクエスト가 사용자로부터가 아니라 선택한 プロキシ 서버의 IPアドレス 및 위치에서 오는 것으로 인식합니다.

Invoke-WebRequest로 PowerShell プロキシ를 사용하기 시작하려면, プロキシ 서버 URL이 어떤 형태인지 이해해야 합니다.

다음은 PowerShell Invoke-WebRequest プロキシ의 URL입니다:

```powershell
<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]
```

**구성 요소는 다음과 같습니다:**

- `PROTOCOL`: プロキシ 서버에 연결하는 데 사용할 프로토콜입니다.
- `HOST`: プロキシ 서버 호스트네임의 IPアドレス 또는 URL입니다.
- `PORT`: プロキシ 서버가 수신 대기하는 포트 번호입니다.
- `USERNAME`: プロキシ 認証에 사용하는 선택적 사용자 이름입니다.
- `PASSWORD`: プロキシ 認証에 사용하는 선택적 비밀번호입니다.

> 💡 **중요:**\
> **<PROTOCOL>:// 부분은 Invoke-WebRequest에서 필수입니다. 이를 생략하면 リクエスト는 다음 오류로 실패합니다:**\
> **Invoke-WebRequest : This operation is not supported for a relative URI.**

PowerShell 5.1에서 Invoke-WebRequest는 HTTP만 지원하는 반면, PowerShell 7.x에서는 HTTPS와 SOCKS도 지원합니다.

이제 유효한 HTTP プロキシ를 가져올 시간입니다!

예를 들어 온라인에서 무료로 다음과 같은 값을 찾을 수 있습니다:

```
Protocol: HTTP; IP Address: 190.6.23.219; Port: 999
```

이를 다음처럼 조합합니다:

```
http://190.6.23.219:999
```

>⚠️ **경고:**\
> **무료 プロキシ는 신뢰할 수 없고, 오류가 잦고, 느리며, 데이터를 과도하게 수집하고, 수명이 짧습니다. 사용하지 마십시오!**

해결책은? 시장 최고의 제공업체인 Bright Data의 프리미엄 プロキシ입니다. 구독하고 신뢰할 수 있는 プロキシ를 무료로 사용해 보십시오.

[Bright Data의 プロキシ 서비스](https://brightdata.co.kr/proxy-types)는 認証으로 보호되어 신뢰할 수 있는 사용자만 접근할 수 있습니다. 

다음과 같다고 가정해 보겠습니다:

Protocol: `HTTP`
Host: `45.103.203.109`
Port: `9571`
Username: `admin-4521`
Password: `rUuH3tJqf`

그렇다면 Invoke-WebRequest プロキシ URL은 다음과 같습니다:

```powershell
http://admin-4521:@rUuH3tJqf45.103.203.109:9571
```

## How to Specify an HTTP Proxy in Invoke-WebRequest

시작하기 전에 다음을 실행하십시오:

```powershell
Invoke-WebRequest "https://httpbin.org/ip"
```

다음과 같이 출력될 수 있습니다:

```yaml
StatusCode         : 200
StatusDescription  : OK
Content           : {
                      "origin": "194.34.233.12"
                    }
RawContent        : HTTP/1.1 200 OK
                    Connection: keep-alive
                    Access-Control-Allow-Origin: *
                    Access-Control-Allow-Credentials: true
                    Content-Length: 32
                    Content-Type: application/json
                    Date: Thu, 01 Feb 2024 10:46:14 GMT...
Forms            : {}
Headers          : {[Connection, keep-alive], [Access-Control-Allow-Origin, *], 
                   [Access-Control-Allow-Credentials, true], [Content-Length, 32]...}
Images           : {}
InputFields      : {}
Links            : {}
ParsedHtml       : mshtml.HTMLDocumentClass
RawContentLength : 32
```

**Content** 필드에 주목하십시오. 여기에는 **사용자 IP**가 들어 있습니다.

왜일까요? `https://httpbin.org/ip`는 リクエスト의 origin IP를 반환하기 때문입니다. 즉, プロキシ가 설정되지 않았다면 사용자의 머신 IP가 됩니다.

Content 필드만 원한다면 다음을 사용하십시오:

```powershell
$response = Invoke-WebRequest "https://httpbin.org/ip"
$response.Content
```

다음과 같은 내용이 출력됩니다:

```json
{
  "origin": "194.34.233.12"
}
```

해당 リクエスト를 プロキシ를 통해 라우팅하면, 대신 プロキシ 서버 IP가 표시됩니다. 이는 PowerShell Invoke-WebRequest가 실제로 プロキシ를 사용하고 있는지 확인하는 훌륭한 테스트입니다.

Invoke-WebRequest에서 PowerShell プロキシ를 설정하는 방법은 몇 가지가 있습니다:

## Using a Command Line Option

Invoke-WebRequest는 `-Proxy` 플래그를 제공합니다:

```powershell
Invoke-WebRequest -Proxy "<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]" <Uri>
```

따라서 다음과 같습니다:

```powershell
Invoke-WebRequest -Proxy "http://190.6.23.219:999" "https://httpbin.org/ip"

Invoke-WebRequest -Uri "http://httpbin.org/ip" `
  -Proxy "http://brd.superproxy.io:33335" `
  -ProxyCredential (
    New-Object System.Management.Automation.PSCredential(
      "brd-customer-CUSTOMER_ID-zone-ZONE’S_NAME",
      ("ZONE’S_PASSWORD" | ConvertTo-SecureString -AsPlainText -Force)
    )
  )
```

결과는 다음과 같습니다:

```yaml
StatusCode         : 200
StatusDescription  : OK
Content           : {
                      "origin": "190.6.23.219"
                    }
RawContent        : HTTP/1.1 200 OK
                    Connection: keep-alive
                    Access-Control-Allow-Origin: *
                    Access-Control-Allow-Credentials: true
                    Content-Length: 31
                    Content-Type: application/json
                    Date: Thu, 01 Feb 2024 12:36:56 GMT...
Forms            : {}
Headers          : {[Connection, keep-alive], [Access-Control-Allow-Origin, *], 
                   [Access-Control-Allow-Credentials, true], [Content-Length, 31]...}
Images           : {}
InputFields      : {}
Links            : {}
ParsedHtml       : mshtml.HTMLDocumentClass
RawContentLength : 31
```
여기서 `origin`이 プロキシ 서버 IP와 일치하므로, リクエスト가 プロキシ를 통해 처리되었음을 보여줍니다. 완벽합니다!

> **Note:** 무료 プロキシ는 수명이 짧습니다. 해당 プロキシ가 실패하면 다른 것을 선택하십시오.

## Using Environment Variables

PowerShell 7.0부터 Invoke-WebRequest는 환경 변수를 통한 プロキシ 구성을 지원합니다.

두 개의 env를 설정하십시오:

- `HTTP_PROXY:` HTTP リクエスト에 대한 プロキシ URL입니다.
- `HTTPS_PROXY:` HTTPS リクエスト에 대한 プロキ시 URL입니다.

Windows에서:

```powershell
$env:HTTP_PROXY  = "<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]"
$env:HTTPS_PROXY = "<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]"
```

예를 들어:

```powershell
$env:HTTP_PROXY  = "http://190.6.23.219:999"
$env:HTTPS_PROXY = "http://190.6.23.219:999"
```

macOS / Linux에서:

```bash
export HTTP_PROXY="<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]"
export HTTPS_PROXY="<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]"
```

예를 들어:

```bash
export http_proxy="http://190.6.23.219:999"
export https_proxy="http://190.6.23.219:999"
```

이제 모든 Invoke-WebRequest 호출은 자동으로 이 プロキシ들을 통해 처리됩니다. 다음을 실행하십시오:

```powershell
Invoke-WebRequest "https://httpbin.org/ip"
```

`origin`에서 다시 プロキ시 IP가 표시될 것입니다. 완벽합니다.

이 プロキ시를 끄려면, 값을 해제하십시오:

```powershell
$env:HTTP_PROXY  = ""
$env:HTTPS_PROXY = ""
```

macOS / Linux:

```bash
unset HTTP_PROXY
unset HTTPS_PROXY
```

그런 다음 Invoke-WebRequest `"https://httpbin.org/ip"`는 다시 사용자 IP를 보여줍니다.

## How To Use HTTPS and SOCKS Proxies in PowerShell

HTTPS 또는 [SOCKS プロキシ](https://brightdata.co.kr/solutions/socks5-proxies)를 사용하려면 PowerShell 7.x+를 사용해야 합니다. 그렇지 않으면 다음 오류가 발생합니다:

```
Invoke-WebRequest : The ServicePointManager does not support proxies with the https scheme.
```

또는 SOCKS의 경우:

```
Invoke-WebRequest : The ServicePointManager does not support proxies with the socks scheme.
```

PowerShell 7.x에서도 구조는 동일합니다:

```powershell
Invoke-WebRequest -Proxy "<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]" <Uri>
```

다만 `<PROTOCOL>`은 `https`, `socks4`, `socks4a`, `socks5`, `socks5a`(기존 `http`뿐만 아니라)일 수 있습니다.

이들 외의 프로토콜을 시도하면 다음 오류가 발생합니다:

```
Invoke-WebRequest: Only the 'http', 'https', 'socks4', 'socks4a' and 'socks5' schemes are allowed for proxies.
```

SOCKS プロキシ 예시는 다음과 같습니다:

```powershell
Invoke-WebRequest -Proxy "socks5://94.14.109.54:3567" "http://httpbin.org/ip"
```

출력은 다음과 같을 수 있습니다:

```yaml
StatusCode         : 200
StatusDescription  : OK
Content           : {
                      "origin": "94.14.109.54"
                    }
RawContent        : HTTP/1.1 200 OK
                    Connection: keep-alive
                    Access-Control-Allow-Origin: *
                    Access-Control-Allow-Credentials: true
                    Content-Length: 31
                    Content-Type: application/json
                    Date: Thu, 01 Feb 2024 12:47:56 GMT...
Forms            : {}
Headers          : {[Connection, keep-alive], [Access-Control-Allow-Origin, *],
                   [Access-Control-Allow-Credentials, true], [Content-Length, 31]...}
Images           : {}
InputFields      : {}
Links            : {}
ParsedHtml       : mshtml.HTMLDocumentClass
RawContentLength : 31
```

## Tips and Tricks You Need to Know

프로처럼 PowerShell Invoke-WebRequest プロキシ를 다루는 데 유용한 트릭을 확인하십시오.

### Ignore the PowerShell Proxy Configuration

구성된 환경 변수 プロキシ를 건너뛰려면 `-NoProxy`를 사용하십시오:

```powershell
Invoke-WebRequest -NoProxy <Uri>
```

이는 プロキ시 없이 `<Uri>`에 연결합니다.

검증하려면 envs에 プロキ시를 설정한 다음 다음을 실행하십시오:

```powershell
Invoke-WebRequest -NoProxy "https://httpbin.org/ip"
```

프로キ시 IP가 아니라 사용자 IP가 표시될 것입니다.

### Avoid SSL Certificate Errors

HTTP プロキ시 사용 시 SSL 인증서 오류로 실패할 수 있습니다. -SkipCertificateCheck를 사용하십시오:

```powershell
Invoke-WebRequest -SkipCertificateCheck -Proxy "<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]" <Uri>
```

`-SkipCertificateCheck`는 안전하지 않은 서버 연결을 허용합니다. 보안상 안전하지 않으므로 신뢰할 수 있는 호스트에서만 사용해야 합니다.

예시:

```powershell
Invoke-WebRequest -SkipCertificateCheck -Proxy "http://190.6.23.219:999" "https://httpbin.org/ip"
```

## Which PowerShell Proxy Should You Use?

Invoke-WebRequest 목표에 따라 달라집니다. 주요 プロキ시 유형을 고려하십시오:

- [**データセンタープロキシ:**](https://brightdata.co.kr/proxy-types/datacenter-proxies) 빠르고 저렴하지만, 식별되면 쉽게 차단될 수 있습니다.
- [**レジデンシャルプロキシ:**](https://brightdata.co.kr/proxy-types/residential-proxies) 디바이스에서 제공되는 실제 IPアドレス를 ローテーティング합니다. 지역 차단 콘텐츠 또는 アンチボット 회피에 적합합니다.
- [**ISPプロキシ:**](https://brightdata.co.kr/proxy-types/isp-proxies) ISP가 제공하는 안전하고 빠른 スタティックプロキ시 IP—SEO 또는 시장 조사에 이상적입니다.
- [**モバイルプロキシ:**](https://brightdata.co.kr/proxy-types/mobile-proxies) 실제 모바일 디바이스 기반으로, 모바일 전용 앱/사이트에 최적입니다.

자세한 내용은 [プロキシ IP 유형 가이드](https://brightdata.co.kr/blog/proxy-101/ultimate-guide-to-proxy-types)에서 확인하십시오.