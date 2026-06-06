+++
title = "방치된 polyfill.io가 띄운 HTTP Basic Auth 팝업 — 자격증명 탈취 공급망 공격"
date = 2026-06-03T17:00:00+09:00
draft = false
tags = ["polyfill", "supply-chain", "http-basic-auth", "burp-suite", "credential-leak"]
categories = ["security"]
description = "방치된 polyfill.io 스크립트가 띄우는 HTTP Basic Auth 팝업을 공급망 공격(supply-chain attack) 관점에서 분석하고, 입력한 자격증명이 실제로 외부 서버로 전송되는지를 Burp Suite로 직접 검증한 보안 연구 기록이다."
+++

여러 기술 블로그에서 동일하게 관측된 `polyfill.io` 스크립트가 띄우는
HTTP Basic Auth 팝업. 이를 **신뢰된 서드파티 의존성이 변질되어 방문자의 자격증명을
노리는 공급망 공격(supply-chain attack)** 의 한 형태로 보고, 입력한 자격증명이 실제로
외부로 전송되는지, 공급망 공격 체인에 관해 직접 검증·분석한 기록이다.

<!--more-->

## TL;DR

- 여러 GitHub Pages / Tistory 기술 블로그를 둘러보던 중, 일부 페이지에서 `https://polyfill.io`를 출처로 하는 **HTTP Basic Auth 로그인 팝업**이 뜨는 것을 발견했다.
- 원인은 블로그 테마에 남아있던 `<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>` 한 줄이었다.
- 현재 `polyfill.io`는 **사이트 전체가 `401 Authorization Required`로 잠겨 있고**, 어떤 자격증명을 넣어도 통과되지 않는다.
- 하지만 **사용자가 입력한 ID/PW는 `Authorization: Basic base64(id:pw)` 헤더로 polyfill.io까지 그대로 전송된다.** base64는 암호화가 아니라 단순 인코딩이다.
- 서버 도달 시점에 자격증명은 **평문**이며, 탈취 이력이 있는 도메인이므로 **절대 실제 자격증명을 입력해서는 안 된다.**
- 이는 사이트 운영자가 직접 심은 코드가 아니라 **신뢰하던 외부 CDN이 변질되며 방문자에게 전가된 위험**으로, 전형적인 웹 공급망 공격의 양상을 띤다.
- 추적 결과 이 라인의 근원은 **MathJax의 옛 설치 안내**였고, 인기 Jekyll 테마와 **Tistory 공식 #1 스킨의 구버전**에 박제되어 퍼졌다. 이미 패치됐지만, **과거 버전을 적용한 채 유지한 블로그**에 위험이 남아있다.
- "왜 지금 갑자기"의 답은 — 블로그 코드는 그대로인데 **polyfill.io 서버 응답이 `200`→`401`로 바뀐 것**이 트리거였다.

---

## 1. 발견 경위

특별한 동기가 있었던 건 아니었다. 평소처럼 GitHub Pages와 Tistory에 올라온
기술 블로그 글들을 찾아보던 중, **몇몇 블로그에 접속하면 페이지 위로 낯선 로그인
팝업이 뜨는 것**을 발견했다.

팝업의 출처는 페이지 본문이 아니라 `https://polyfill.io` 였다.
브라우저가 띄운 네이티브 HTTP Basic Auth 인증창이었다.

<!-- 📷 스크린샷 배치: 블로그 위에 뜬 polyfill.io 출처의 HTTP Basic Auth 로그인 팝업
     (사용자 이름/비밀번호 입력창 + "https://polyfill.io" 출처가 보이는 화면) -->

![polyfill.io HTTP Basic Auth 팝업](./images/polyfill_basic_auth.png)

"왜 블로그 글을 보려는데 polyfill.io가 로그인을 요구하지?"

일반 사용자라면 무심코 자신의 계정 정보를 입력했을 법한 화면이다. 그러나 이 팝업의
출처 도메인은 과거 대규모 공급망 사고로 악명이 높았던 바로 그 `polyfill.io`였다.
서드파티 스크립트가 인증을 요구한다는 것 자체가 비정상 신호였기에, **이를 자격증명
수집을 노린 공급망 공격으로 가정하고 실제 자격증명이 외부로 유출되는지를 검증하기로
했다.**

---

## 2. 배경 — polyfill.io 공급망 사건

`polyfill.io`는 원래 구형 브라우저 호환성을 위한 정상적인 무료 polyfill CDN이었다.
그러나 **2024년, `cdn.polyfill.io` 도메인이 제3자에게 매각된 뒤** 방문자를 악성/도박
사이트로 리다이렉트하고 악성 스크립트를 주입하는 정황이 다수 보고되었다(Sansec,
Cloudflare 등 공개 경고). 수십만 개 사이트가 영향을 받았고, 안전한 미러(Cloudflare,
Fastly)로의 이전이 권고되었다.

### 왜 이것이 공급망 공격인가

여기서 핵심은 **개별 사이트가 해킹당한 것이 아니라는 점**이다. 어떤 블로그도
침해당하지 않았고, 운영자가 악성 코드를 심지도 않았다. 단지 과거에 신뢰하고
연결해 둔 **외부 의존성(CDN)이 공격자의 손에 넘어가면서**, 그 의존성을 불러오던
수많은 사이트가 동시에 공격 경로로 바뀐 것이다. 이것이 웹 공급망 공격의 본질이다.

```
[정상 시점]
  방문자 ──► 블로그 ──(신뢰)──► polyfill.io ──► 정상 polyfill.js

[도메인 탈취 후]
  방문자 ──► 블로그 ──(여전히 신뢰)──► polyfill.io(공격자) ──► 악성 행위
                                                          (리다이렉트 / 악성 JS / 자격증명 유도)
```

서드파티 `<script>`는 한번 삽입되면 **해당 도메인에 내 페이지의 실행 권한을 영구
위임**하는 것과 같다. 그 도메인이 매각·탈취되는 순간, 위임했던 권한은 그대로
공격자에게 넘어간다. 방문자 입장에서는 **자신이 신뢰한 블로그가 띄운 화면**으로
보이기 때문에 의심 없이 자격증명을 입력하게 된다 — 공격자가 노리는 것이 바로 이
"전이된 신뢰(transitive trust)"다.

### 위험이 장기간 잠복하는 이유

더 큰 문제는 **사고 이후에도 과거에 박아둔 `<script>` 태그가 그대로 남아있는
블로그가 많다는 것**이다. 운영자는 자신의 사이트가 위험을 전파하고 있다는 사실을 모르고 있는 경우가 많았다.
정적 사이트 생성기(Jekyll, Hugo 등)의 테마에 포함되어
있으면, 글을 한 편도 새로 쓰지 않아도 모든 페이지가 자동으로 이 스크립트를 계속
로드한다. 내가 방문한 블로그들도 그랬다.

![블로그 소스코드에 삽입된 polyfill.io 스크립트 태그](./images/blog_source_code.png)

```html
<!-- 테마에 남아있던 문제의 한 줄 -->
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
```

이 스크립트를 로드하려는 순간 polyfill.io가 `401 + WWW-Authenticate: Basic`을 응답하면서
브라우저가 네이티브 인증 팝업을 자동으로 띄운 것이다.

---

## 3. 검증 목표

> **"이 팝업에 입력한 자격증명이 실제로 외부(polyfill.io)로 전송되는가?"**

이걸 추측이 아니라 요청 레벨에서 확정하는 것이 목표였다.

핵심은 HTTP Basic Auth의 동작 방식에 있다. 사용자가 팝업에 값을 입력하면,
브라우저는 **같은 URL로 `Authorization` 헤더를 붙여 재요청**한다.

```
[1차 요청]  GET /v3/polyfill.min.js          (Authorization 없음)
            ↓
[응답]      401 + WWW-Authenticate: Basic     ← 여기서 팝업이 뜸
            ↓
[사용자가 팝업에 id:pw 입력]
            ↓
[2차 요청]  GET /v3/polyfill.min.js
            Authorization: Basic base64(id:pw)   ← 값은 여기에 실려 전송된다
```

즉 자격증명은 **1차 요청이 아니라 입력 이후의 2차 요청**에 실린다.
이 2차 요청을 Burp Suite로 직접 잡아 증명했다.

---

## 4. 현재 polyfill.io의 상태

확인 결과 `polyfill.io`는 Cloudflare 뒤에 nginx 오리진이 있는 구조였고,
**사이트 전체가 Basic Auth로 잠겨 있었다.**

- 어떤 경로로 요청해도 `401 Authorization Required` (nginx)
- 더미 자격증명을 넣어도 동일하게 거부됨

즉 "올바른 비밀번호를 맞히면 통과시키는 피싱 함정형"이 아니라,
**도메인 전체가 잠긴 전면 차단형**라고 판단할 수 있다. 정상 polyfill JS도, 악성 JS도 내려오지 않는다.

> 이 글에서는 접속자가 실수로 따라 들어가지 않도록 구체적인 IP와 일부 식별 정보는 가렸다.

---

## 5. 증명 — Burp Suite로 자격증명 전송 확인

### 5-1. Authorization 헤더로 전송되는 요청

Burp Repeater로 팝업 입력과 동일한 요청을 재현했다.
더미 자격증명으로 `test1234:213124213`을 사용했다.

> ⚠️ 검증 중에도 **절대 진짜 자격증명을 입력하지 않는다.** 더미 값만 사용한다.

![Burp Request/Response — Authorization 헤더와 401 응답](./images/burp_request_response.png)

**Request 패널 (왼쪽)** 에서 핵심은 3번 줄이다.

```http
GET /v3/polyfill.min.js?features=es6 HTTP/2
Host: polyfill.io
Authorization: Basic dGVzdDEyMzQ6MjEzMTI0MjEz      ← 입력값이 실려 전송됨
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) ... Chrome/146.0.0.0 Safari/537.36
Sec-Fetch-Dest: script
Referer: https://[REDACTED].github.io/
```

`Sec-Fetch-Dest: script`, `Sec-Fetch-Mode: no-cors`에서 보이듯,
이 요청은 블로그가 `<script>`로 polyfill.io를 로드하는 바로 그 흐름이다.
그리고 **`Authorization: Basic dGVzdDEyMzQ6MjEzMTI0MjEz` 헤더로 자격증명이 실려 나간다.**

**Response 패널 (오른쪽)** 은 다음과 같다.

```http
HTTP/2 401 Unauthorized
Server: cloudflare
Www-Authenticate: Basic realm="Restricted Area"
...
<html>
<head><title>401 Authorization Required</title></head>
<body>
<center><h1>401 Authorization Required</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

Cloudflare 엣지를 거쳐 nginx 오리진이 `401`을 돌려준다.
**중요한 것은 "거부됐다"가 아니라, 그 거부 응답을 받기까지 내 자격증명이
이미 polyfill.io 서버까지 도달했다는 사실이다.**

### 5-2. base64는 암호화가 아니다 — Decoder로 즉시 복원

전송된 `Authorization` 헤더 값 `dGVzdDEyMzQ6MjEzMTI0MjEz`를
Burp Decoder에 넣어 base64 디코딩하면 입력값이 그대로 나온다.

![Burp Decoder — base64가 평문 자격증명으로 즉시 복원됨](./images/credentials_base64_decode.png)

```
dGVzdDEyMzQ6MjEzMTI0MjEz   →   test1234:213124213
```

Basic Auth의 base64는 **암호화가 아니라 단순 인코딩**이다.
누구든 즉시 평문으로 되돌릴 수 있다. TLS 터널 안에서는 보호되지만,
터널이 끝나는 지점(Cloudflare 엣지, nginx 오리진)에서는 **완전한 평문**이다.

---

## 6. 위협 모델 — 진짜 위험은 무엇인가

```
사용자: "polyfill.io가 로그인을 요구하네? 내 계정 비번 넣어볼까?"
   │
   ▼  브라우저가 Authorization: Basic base64(id:pw) 전송
Cloudflare 엣지 (TLS 종료 지점) ──► 여기서 평문 id:pw 노출
   │
   ▼
방치/탈취된 polyfill.io nginx 오리진 ──► access log에 평문 적재 가능
   │
   ▼
공격자: 수집된 자격증명으로 다른 서비스에 credential stuffing
```

핵심 포인트:

1. **신뢰의 전이가 공격의 시작점이다.** 방문자는 polyfill.io를 신뢰한 적이 없다.
   방문자가 신뢰한 것은 블로그이고, 블로그가 신뢰한 것이 polyfill.io다. 공급망
   공격은 이 신뢰 사슬의 가장 약한 고리(탈취된 CDN)를 끊어, 방문자가 자신도 모르게
   공격자를 신뢰하도록 만든다.
2. **TLS는 전송 구간만 보호한다.** 터널이 끝나는 지점에서 자격증명은 평문이다.
3. **로깅은 서버 측 재량이며 매우 쉽다.** nginx는 로그 포맷에 `$http_authorization`
   한 줄만 추가하면 모든 입력 자격증명을 남길 수 있다. 외부에서 "로깅 여부"는
   검증 불가능하지만, **하려면 한 줄**이고 탈취 이력 도메인이므로 충분히 의심스럽다.
4. **무한 401이 오히려 위험을 키운다.** "어? 안 되네, 다른 비번도?" 하며 사용자가
   여러 개의 진짜 자격증명을 연달아 입력하도록 유도하는 효과가 있다.

> 진짜 위험은 "서버가 비번을 검증한다"가 아니라,
> **사용자가 습관적으로 자기 진짜 비밀번호를 탈취 도메인에 헌납한다**는 것이다.

---

## 7. 공급망 공격으로서의 함의

이번 사례는 비교적 단순한 Basic Auth 자격증명 유도였지만, **웹 공급망 공격이
가진 구조적 위험**을 압축해서 보여준다. 보안 연구 관점에서 짚어둘 점은 다음과 같다.

- **공격 표면이 내 코드 밖에 있다.** 아무리 내 사이트를 안전하게 작성해도, 불러오는
  서드파티 스크립트 한 줄이 통제권 밖에서 변질되면 방어선이 무력화된다. 코드 리뷰나
  정적 분석으로는 "지금 이 순간 외부 서버가 무엇을 반환하는가"를 보장할 수 없다.
- **피해가 광범위하고 동시다발적이다.** 단일 도메인 탈취 하나로 그 스크립트를 쓰는
  모든 사이트가 동시에 영향을 받는다. polyfill.io 사건이 수십만 사이트에 번진 것이
  그 증거다. 공격자 입장에서는 **한 번의 침해로 막대한 방문자 풀**을 확보한다.
- **변질 시점을 통제할 수 없다.** 스크립트를 삽입할 당시에는 정상이었더라도, 그
  도메인이 언제 매각·탈취될지는 운영자가 알 수 없다. 즉 **"지금 안전함"이 "앞으로도
  안전함"을 보장하지 않는다.**
- **조건부·은밀한 페이로드가 가능하다.** 실제 polyfill 공급망 공격에서는 모든
  방문자가 아니라 특정 조건(모바일 사용자, 특정 시간대 등)에서만 악성 코드를
  내보내 탐지를 회피했다. 본 사례는 자격증명 유도였지만, 같은 통로로 악성 JS 주입,
  드라이브-바이 다운로드, 세션 탈취 등 무엇이든 배달될 수 있다.

### 운영자가 취해야 할 구조적 방어

개별 도메인을 차단하는 것만으로는 부족하다. 서드파티 의존성 자체를 통제하는 습관이 필요하다.

- **Subresource Integrity(SRI)** — `<script>`에 `integrity` 해시를 명시하면,
  내려온 파일이 해시와 다를 경우 브라우저가 실행을 거부한다. CDN이 변질돼도
  원본과 다른 코드는 실행되지 않는다. (단 버전이 동적으로 바뀌는 polyfill류에는
  적용이 까다로워, 그런 의존성은 셀프 호스팅이 더 안전하다.)
- **Content-Security-Policy(CSP)** — `script-src`를 신뢰하는 출처로 제한하면,
  예기치 않은 외부 도메인으로의 요청·실행을 차단할 수 있다.
- **의존성 최소화 / 셀프 호스팅** — 꼭 필요한 외부 스크립트만 남기고, 가능하면
  자체 인프라나 신뢰 가능한 미러에서 직접 서빙한다.
- **주기적 의존성 점검** — 한번 삽입한 서드파티 출처가 여전히 신뢰 가능한지
  정기적으로 재확인한다. "넣고 잊는(set-and-forget)" 것이 가장 위험하다.

---

## 8. 공급망 체인 추적 — 어디서, 어떻게, 왜 지금

"자격증명이 전송된다"는 검증을 마친 뒤, 진짜 궁금증은 따로 있었다.
**이 polyfill.io 라인은 도대체 어디서 흘러들어왔고, 왜 하필 비교적 최근에야
로그인 팝업이 뜨기 시작했으며, 그 규모는 얼마인가?** 추적해보니 하나의 공통
뿌리에서 여러 플랫폼으로 번진 구조가 드러났다.

### 8-1. 공통 뿌리 — MathJax의 옛 설치 안내

문제의 라인은 거의 항상 바로 아래에 `MathJax`를 동반하고 있었다.

```html
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
```

이 조합은 **MathJax v3가 한때 공식 문서에서 권장하던 설치 스니펫**이다. 즉 누군가
악의로 심은 게 아니라, **수식(LaTeX) 렌더링을 붙이려고 당시 표준 안내를 그대로 복사**한
결과다. 그래서 이 문제는 특정 블로그나 테마 하나의 문제가 아니라, **그 안내를 따른
모든 정적사이트·블로그·문서 생태계에 광범위하게 퍼져 있다.**

추적 근거를 수치로 남겨둔다. GitHub 코드 검색 기준(2026-06-03 조사 시점):

| 검색 쿼리 | 결과 |
|---|---|
| `"polyfill.io" "MathJax-script" in:file` | 약 **5,100건** |
| `"https://polyfill.io/v3/polyfill.min.js?features=es6" filename:mathjax.html` | 약 **196개 저장소** |

대형 문서화 프로젝트인 mkdocs-material도 동일 원인으로 공식 제거 공지를 냈다:
[squidfunk/mkdocs-material#7295](https://github.com/squidfunk/mkdocs-material/issues/7295)
(*"⚠️ Action required: remove polyfill.io in `extra_javascript`"*, 2024-06-25).

### 8-2. 경로 ① — GitHub Pages / Jekyll 테마

GitHub에 공개된 블로그들은 출처 추적이 가능했다. 영향받은 블로그들은 인기 Jekyll
테마 하나를 공유하고 있었다 — `vszhub/not-pure-poole`
([github.com/vszhub/not-pure-poole](https://github.com/vszhub/not-pure-poole),
⭐146 / 🍴485 fork). 그 테마의 `_includes/mathjax.html` **단 한 파일**에 polyfill
라인이 박혀 있었다.

**추적 근거 (커밋 단위):**

- 문제의 라인을 도입한 커밋:
  [`bfc3b6a16360cd8c75fe60f271b7971179923d48`](https://github.com/vszhub/not-pure-poole/commit/bfc3b6a16360cd8c75fe60f271b7971179923d48)
  — *"Import MathJax"* (vszhub, **2020-10-02**)
- 해당 파일의 커밋 시점 스냅샷(permalink, 19행에서 polyfill 확인 가능):
  [`/blob/bfc3b6a.../_includes/mathjax.html`](https://github.com/vszhub/not-pure-poole/blob/bfc3b6a16360cd8c75fe60f271b7971179923d48/_includes/mathjax.html)
- 이 `mathjax.html`은 **위 커밋 한 번으로 생성된 뒤 한 번도 수정되지 않았다**
  (`git log -- _includes/mathjax.html` 결과 커밋이 단 하나).
- 같은 커밋이 함께 추가한 파일 목록:
  `_includes/mathjax.html`, `_layouts/default.html`, `_posts/2020-10-02-testing-mathjax.md`, `README.md`.
  → **"Testing MathJax" 샘플 글까지 함께 배포**됐기 때문에, 테마를 fork한 블로그에는
  그 글이 자동으로 딸려오고, 그 글이 항상 polyfill을 로드한다.
- 저장소는 **2020-10-09에 아카이브(읽기 전용)** 되어 중앙 패치가 불가능하고, 485개의
  fork가 각자 알아서 고쳐야 하는 상태였다.

> 정리: GitHub 쪽은 커밋 해시까지 짚어가며 **언제·무엇이 들어왔는지 정확히 추적**할 수
> 있었다. "한번 복사된 뒤 동결된 의존성"이 위험의 온상이라는 점이 그대로 드러난다.

### 8-3. 경로 ② — Tistory 공식 스킨 (그리고 더 흥미로운 사실)

추적 중 동일한 팝업이 **Tistory 블로그에서도 발생**한다는 제보를 접했다. 스킨 편집
화면을 확인하자 결정적인 사실이 나왔다 — 문제의 스킨은 개인 제작물이 아니라
**Tistory가 공식 제공하는 기본 스킨**이었고, 그 HTML의 head에 동일한 polyfill 라인이
있었다.

![Tistory 공식 #1 스킨 정보 — 제작자 Tistory](./images/skin_info.png)

![구버전 #1 스킨 HTML — 19행 polyfill.io 라인 존재](./images/tistory_skin_1_legacy_html.png)

여기서 **결정적인 실험**을 했다. 같은 "#1" 스킨을 내 테스트 블로그에 **새로 적용**한
뒤 HTML을 열어본 것이다. 결과는 다음과 같다.

![최신 #1 스킨 HTML — polyfill.io 제거 + jQuery SRI 적용](./images/tistory_skin_1_html.png)

| 항목 | 구버전 스킨 (취약) | 최신 스킨 (패치됨) |
|---|---|---|
| `polyfill.io` `<script>` | ✅ 존재 → 취약 | ❌ **제거됨** |
| `MathJax-script` | ✅ 존재 | ❌ 함께 제거됨 |
| jQuery 3.x | `integrity` 없음 | ✅ **SRI 추가** (`integrity="sha256-..."` + `crossorigin`) |

→ **Tistory는 이미 스킨을 패치했다.** 최신 버전에는 polyfill.io가 없고, 오히려 jQuery에
**SRI(Subresource Integrity)** 까지 추가되어 보안이 강화돼 있었다. 그렇다면 왜 아직도
팝업이 뜨는 블로그가 있는가?

**핵심은 Tistory 스킨의 동작 방식이다.** 스킨을 적용하는 순간 그 시점의 HTML이
블로그에 **복사·고정** 되며, 이후 Tistory가 원본 스킨을 패치해도 **이미 적용한
블로그에는 적용되지 않는다.** 즉 취약한 블로그 = **과거 polyfill 포함 버전의 스킨을
적용한 뒤, 재적용·업데이트하지 않은 블로그**다.

**추적 근거와 그 한계:**

- 스킨 정보 패널에 표기된 출처: 제작자 `Tistory <tistoryblog@daum.net>`, 저작권
  `MIT License` (위 스킨 편집 화면 캡처) → 개인이 아닌 **플랫폼 공식 제공 스킨**임이 명시됨.
- Tistory 공식 GitHub 조직에 공개된 구 테마(`tistory/tistory-theme-ray`,
  [ray](https://github.com/tistory/tistory-theme-ray) /
  [ray2](https://github.com/tistory/tistory-theme-ray2), 2018-02)에는 polyfill+MathJax
  라인이 **없다.** 즉 이 라인은 **그 이후 MathJax 지원을 추가하면서 유입**된 것으로 보인다.
- 다만 현행 #1 스킨은 **플랫폼 내부 스킨**이라 공개 git 저장소가 없다. Jekyll처럼
  **커밋 해시로 도입 시점을 못 박는 것은 불가능**하며, 도입 시점은 "ray(2018) 이후 ~
  현재 패치 이전" 구간으로만 좁힐 수 있다(정황 추정).
- 같은 이유로 GitHub 코드 검색도 **0건**이라(모든 Tistory 치환자 시그니처 조합으로
  시도) **정확한 영향 규모는 외부에서 셀 수 없다.** 다만 오염원이 **플랫폼 공식 기본
  스킨**이라는 점에서, 잠재 규모는 GitHub 사례를 상회할 수 있다.

> Jekyll(공개 git → 커밋 단위 추적 가능)과 Tistory(내부 스킨 → 정황 추정만 가능)의
> 이 비대칭이, 같은 위협이라도 플랫폼에 따라 추적 가능성이 얼마나 달라지는지를 보여준다.

### 8-4. "왜 지금 갑자기" — 트리거의 정체

가장 흥미로운 질문의 답은 이것이다. **블로그 코드는 수년 전 그대로인데, 왜 하필
최근에 팝업이 뜨기 시작했는가?** 바뀐 것은 블로그가 아니라 **polyfill.io 서버의
응답**이었다.

```
[~2024-02 이전]  polyfill.io → 200 OK (아마도 정상 JS)      → 팝업 없음 (조용히 동작)
[2024-02 ~ ]     도메인 탈취 → 악성 리다이렉트/주입    → 일부 악성 동작
[현재]           polyfill.io → 전 경로 401 Basic Auth  → ★ 브라우저가 로그인 팝업 ★
```

클라이언트(블로그)는 손 하나 대지 않았는데, **의존하던 외부 서버의 응답이
`200(정상)` → `401(인증 요구)`로 바뀐 순간, 전 세계의 방치된 polyfill 태그가 일제히
로그인 팝업을 띄우기 시작**한 것이다. 이것이 공급망 공격의 가장 무서운 속성이다 —
**공격자는 내 코드를 단 한 줄도 건드리지 않고, 원격에서 모든 피해 사이트의 동작을
한꺼번에 바꿀 수 있다.**

### 8-5. 완성된 공급망 체인

```
① MathJax v3 옛 공식 설치 안내 = polyfill.io + tex-mml-chtml.js   (공통 뿌리)
        │
        ├─► [경로 A] Jekyll 테마의 mathjax.html 에 포함 → fork로 확산 → 테마 동결
        │
        └─► [경로 B] Tistory 공식 #1 스킨에 포함 → 사용자 적용 시 HTML "박제"
        │
        ▼
② 원천(테마/스킨)은 이후 패치·동결됨 — 그러나 이미 복사된 사본은 자동 갱신 안 됨
        │
        ▼
③ 2024-02  polyfill.io 도메인 탈취 (정상 CDN → 공격자 통제)
        │
        ▼
④ 현재  polyfill.io 전 경로 401 → 방치된 사본을 가진 블로그에서 일제히 팝업 발생
        │
        ▼
⑤ 방문자가 입력한 자격증명이 Authorization 헤더로 탈취 도메인까지 전송 (5장에서 실증)
```

한 줄로 요약하면: **"수년 전 정상이던 한 줄의 의존성이, 원천에서 패치된 뒤에도
박제된 사본으로 남아있다가, 2024년 도메인 탈취로 서버 응답이 바뀌자 일제히
자격증명 수집 창구로 깨어난 것."**

---


## 9. 결론 및 권고

분석 결과를 정리하면 다음과 같다.

| 질문 | 결론 |
|---|---|
| 입력 자격증명이 외부로 전송되는가? | ✅ **확정**. `Authorization: Basic base64(id:pw)`로 polyfill.io까지 전송 |
| base64는 안전한가? | ❌ 인코딩일 뿐. Decoder로 즉시 평문 복원 |
| 서버가 자격증명을 검증/통과시키는가? | ❌ 아니오. 어떤 값이든 401 (전면 차단형) |
| 서버가 로깅하는가? | ⚠️ 외부 검증 불가. 단, 한 줄로 가능하며 탈취 이력상 의심됨 |
| 공급망 공격으로 볼 수 있는가? | ✅ 변질된 외부 의존성이 다수 사이트 방문자의 자격증명을 노린 전형적 양상 |
| 어디서 비롯됐는가? | MathJax 옛 설치 안내 → Jekyll 테마 / Tistory 공식 #1 스킨의 **구버전**에 박제 |
| 왜 지금 발생했는가? | 블로그 코드 불변, **polyfill.io 서버 응답이 200→401로 바뀐 것**이 트리거 |

### 내가 영향받는지 1분 만에 확인하기

브라우저에서 내 블로그를 연 뒤 개발자도구 콘솔(F12)에 붙여넣는다.

```js
// true 가 나오면 내 사이트가 polyfill.io 를 로드 중 = 취약
[...document.scripts].some(s => s.src.includes('polyfill.io'))
```

> ⚠️ 주의: Tistory가 플랫폼 차원에서 제공하는 `...daumcdn.net/.../polyfills-legacy.js`
> 는 **정상 스크립트**다. 위험한 것은 호스트가 **`polyfill.io`** 인 `<script>` 뿐이니
> 혼동하지 말 것.

### 블로그/사이트 운영자라면

**Jekyll / Hugo 등 정적사이트** — 테마 소스(`_includes/`, `layouts/partials/head.html`,
사용 중인 테마의 head/mathjax 템플릿)에서 `polyfill.io` script 태그를 **즉시 제거**한다.

```diff
- <script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
+ <script src="https://cdnjs.cloudflare.com/polyfill/v3/polyfill.min.js?features=es6"></script>
```
최신 MathJax v3는 polyfill 없이도 동작하므로, 대부분 위 한 줄을 **삭제만** 하면 된다.

**Tistory 사용자** — 공식 #1 스킨을 쓰는 경우, **스킨을 한 번 재적용(또는 최신 스킨으로
변경)** 하면 polyfill.io가 제거된 패치 버전으로 갱신된다. 직접 고치려면
`꾸미기 → 스킨 편집 → HTML`에서 `polyfill.io` 라인을 찾아 삭제하고 "적용"을 누른다.

### 방문자라면

- 저 팝업에 **어떤 실제 자격증명도 입력하지 말 것.** "취소" 후 페이지를 떠난다.
- uBlock Origin 등 차단기를 쓰면 polyfill.io 요청 자체가 막힌다.
- 이미 진짜 비밀번호를 입력한 적이 있다면 → **그 비밀번호를 쓰는 모든 서비스의 비밀번호를 변경**한다.

### 한 줄 교훈

> **공급망 공격은 내 코드가 아니라 내가 신뢰한 의존성을 통해 들어온다.** 한번 박아둔
> 서드파티 `<script>`는 그 도메인이 탈취되는 순간 내 사이트를 통째로 공격 표면으로
> 바꾸고, 그 위험은 내가 아니라 나를 믿고 들어온 방문자에게 전가된다.
> 그리고 HTTP Basic Auth의 base64는 보안이 아니므로, 신뢰할 수 없는 출처가 띄운
> 인증 팝업은 그 자체가 평문 자격증명 수집 창구가 될 수 있다.

---

## 참고 자료 (References)

**추적 근거 — Jekyll 테마 (`not-pure-poole`)**

- 저장소: <https://github.com/vszhub/not-pure-poole> (⭐146 / 🍴485, 2020-10-09 archived)
- polyfill 도입 커밋: [`bfc3b6a16360cd8c75fe60f271b7971179923d48`](https://github.com/vszhub/not-pure-poole/commit/bfc3b6a16360cd8c75fe60f271b7971179923d48) — *"Import MathJax"* (2020-10-02)
- 파일 스냅샷(permalink): [`_includes/mathjax.html` @ bfc3b6a](https://github.com/vszhub/not-pure-poole/blob/bfc3b6a16360cd8c75fe60f271b7971179923d48/_includes/mathjax.html)

**추적 근거 — Tistory 공식 스킨**

- 공식 구 테마(polyfill 없음, 2018-02): [tistory/tistory-theme-ray](https://github.com/tistory/tistory-theme-ray), [ray2](https://github.com/tistory/tistory-theme-ray2)
- 스킨 출처 표기: 제작자 `Tistory <tistoryblog@daum.net>`, MIT License (스킨 편집 화면 캡처)
- 현행 #1 스킨은 플랫폼 내부 스킨 → 공개 git 부재, 도입 커밋 추적 불가(정황 추정)

**규모 산정 — GitHub 코드 검색 (2026-06-03 조사 시점)**

- `"polyfill.io" "MathJax-script" in:file` → 약 5,100건
- `"https://polyfill.io/v3/polyfill.min.js?features=es6" filename:mathjax.html` → 약 196개 저장소
- Tistory 스킨: 코드 검색 0건 (플랫폼 내부 존재 → 정량화 불가)

**관련 사건·공지**

- polyfill.io 공급망 공격 분석 (Sansec): <https://sansec.io/research/polyfill-supply-chain-attack>
- mkdocs-material 제거 공지: [squidfunk/mkdocs-material#7295](https://github.com/squidfunk/mkdocs-material/issues/7295) (2024-06-25)
- 안전 미러: `https://cdnjs.cloudflare.com/polyfill/`, `https://polyfill-fastly.io/`

> 위 커밋 해시·검색 수치는 조사 시점(2026-06) 기준이며, 저장소 상태나 검색 결과는
> 이후 변동될 수 있다. 커밋 해시는 불변이므로 permalink로 항상 동일 스냅샷을 확인할 수 있다.

---

> **면책 / 윤리 고지.** 본 분석은 실제 자격증명을 입력하지 않고 더미 값(`test1234:...`)만
> 사용했으며, 공개적으로 접근 가능한 응답을 관찰하는 선에서 수행되었다. 특정 블로그를
> 비난할 의도가 없어 식별 정보와 IP는 가렸다. 목적은 방치된 서드파티 의존성의 위험을
> 알리고 운영자·방문자가 스스로를 보호하도록 돕는 데 있다.
