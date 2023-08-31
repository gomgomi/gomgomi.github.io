---
title: "Google CSE(Custom Search Engine) 사용하여 검색결과 가져오기"
categories:
    - web
tags:
    - Web
    - CSE
    - Custom Search Engine
    - Google
    - Programmable Search Engine
---

스터디로 진행하는 프로젝트에서 웹에서 검색되는 결과를 수집하는 기능이 필요하게 되었다.  
흔히 크롤링으로 검색하면 나오는 방법이 있지만, 좀 더 나이스 한 방법을 알아보던 중 흔히 사용하는 검색엔진(e.g. Google)을 이용하는 방법을 알게 되었다.

Google에서는 Programmable Search Engine(또는 Custom Search Engine)이라는 명칭으로 제공되는 API를 제공한다.  
그래서 간단히 가이드 문서([Programmable Search Engine - Guide](https://developers.google.com/custom-search/docs/tutorial/creatingcse?hl=ko))를 보고 확인한 내용을 정리해보았다.

---

# Control Panel에서 Programmable Search Engine 생성
[Control Panel](https://programmablesearchengine.google.com/controlpanel/create)에서 검색 요청을 받아 검색하여 결과를 리턴해줄 검색엔진을 생성한다.

{% include figure image_path="/assets/posts/scrap using google cse/Create search engine.png" alt="검색 엔진 생성" caption="검색 엔진 생성" %}

**검색할 사이트**  
이 부분은 검색될 도메인을 지정해주는 부분이다.  
도메인은 설명에도 나와있듯이 서브도메인, 특정 도메인 등 자유롭게 입력할 수 있다.

**검색 엔진 이름**  
생성할 검색 엔진의 이름을 입력하는 부분이다.  
본인이 원하는 이름으로 입력하면 된다.

위 내용을 입력했으면 '만들기' 버튼을 눌러 검색엔진을 생성한다.

종종 웹 페이지나 블로그에서 내부에 구글 검색 창을 가지고 있는 것들이 있는데, 여기서 생성한 검색 엔진을 페이지에 추가한 형태인 것 같다. ‘검색엔진 수정’에서 검색 범위나 디자인 등을 수정하여 사용할 수 있다.

# API 사용 준비
앞서 생성한 검색엔진에 검색을 요청하기 위해서는 **cx**, **key**, **q** 이렇게 세 가지 파라미터가 필수이다.  
그렇다면 이 세 가지 파라미터 값을 어디서 얻을 수 있는지 확인해보자!

## cx - Search Engine ID
[Control Panel](https://programmablesearchengine.google.com/controlpanel/create) 페이지의 왼쪽 메뉴에서 ‘검색엔진 수정’의 콤보박스를 클릭하고 사용할 검색엔진을 선택한다.  
그러면 선택한 검색엔진에 대한 내용이 오른쪽에 표시되고, 그중 '검색 엔진 ID' 항목에서 cx 값을 확인할 수 있다.

{% include figure image_path="/assets/posts/scrap using google cse/Get engine ID.png" alt="검색엔진 ID 획득" caption="검색엔진 ID 획득" %}

## key - API key
[Custom Search JSON API: Introduction](https://developers.google.com/custom-search/v1/introduction?hl=ko) 문서에서 API key를 획득할 수 있다.

위 페이지로 이동하면 눈에 딱 띄는 파란색 ‘Get a Key’ 버튼이 있다.  
이 버튼을 누르면 프로젝트를 선택하라고 하는데, 기존에 생성했던 프로젝트가 있으면 선택하면 되고 없다면 새로 생성하면 된다.

프로젝트를 선택하거나 생성하고 나면 API key가 생성된다.  
API key는 password 개념이라서 보안에 유의해야 한다.

{% include figure image_path="/assets/posts/scrap using google cse/Get API key - step1.png" alt="API 키 획득 - Step 1" caption="API 키 획득 - Step 1" %}

{% include figure image_path="/assets/posts/scrap using google cse/Get API key - step2.png" alt="API 키 획득 - Step 2" caption="API 키 획득 - Step 2" %}

{% include figure image_path="/assets/posts/scrap using google cse/Get API key - step3.png" alt="API 키 획득 - Step 3" caption="API 키 획득 - Step 3" %}

## q - Query
q 파라미터는 간단히 검색어라고 생각하면 된다.  
다른 파라미터도 궁금하다면 [Method: cse.list](https://developers.google.com/custom-search/v1/reference/rest/v1/cse/list?hl=ko#request)에서 확인할 수 있다.

# API 사용하여 결과 획득
API를 사용할 준비가 되었으니, 이제 사용을 해보자!  
직접 코드를 작성하거나 curl 툴을 사용해도 되지만 구글에서는 쉽게 테스트할 수 있도록 웹에서 툴을 제공하고 있다.

[Custom Search JSON API: Introduction](https://developers.google.com/custom-search/v1/introduction?hl=ko#try_it) 문서의 가장 아랫부분에 “Try this API” tool을 클릭하면 테스트할 수 있는 페이지로 이동한다.

{% include figure image_path="/assets/posts/scrap using google cse/Link for API test.png" alt="API 테스트 툴 링크" caption="API 테스트 툴 링크" %}

페이지의 왼쪽 파라미터 목록 중 cx와 q 파라미터 값을 입력해준다. key 값은 API key를 획득했으면 자동으로 넣어주는 듯하다.  
각 파라미터 값을 입력하고 파라미터 입력 아랫부분에 ‘EXECUTE’ 버튼을 누르면 요청 결과를 오른쪽에서 확인할 수 있다.

{% include figure image_path="/assets/posts/scrap using google cse/Use API test tool.png" alt="API 테스트 툴 사용" caption="API 테스트 툴 사용" %}

검색 결과는 response body로 JSON 포맷으로 수신되며 “items”에 해당하는 내용이 검색된 결과 값들이다.  
이 중 각자 필요한 필드를 사용하면 된다.

---

간단히 동작을 확인해보았는데, 소스 코드로 구현하는 경우 Get 요청으로 API를 사용하면 된다.  
그리고 주의할 점은 **하루에 100개의 query 만 무료로 요청**할 수 있다.  
물론 결제하면 더 요청할 수 있다.