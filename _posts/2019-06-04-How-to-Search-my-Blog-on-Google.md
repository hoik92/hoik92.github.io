---
layout: post
title: "[Jekyll] github 블로그 google에 검색 노출시키기"
author: Hoik Jang
categories: jekyll
comments: true
sitemap:
  changefreq: daily
  priority: 1.0
---

블로그는 만들었는데 정작 구글에 검색해도 내 포스팅은 보이지 않는다.

나만 내 블로그의 URL을 알고 나만 볼 것이면 검색엔진에 내 블로그가 노출되지 않아도 상관없지만, 누군가는 내 포스팅으로 도움이 되었으면 한다면 검색엔진에 내 블로그가 노출되도록 만들어보자.

이 포스트는 [JinyongJeong님의 블로그](<http://jinyongjeong.github.io/2017/01/13/blog_make_searched/>) 글에서 배워 내 방식으로 쓴 것이다.

검색엔진의 종류는 다양하지만 개발자가 가장 많이 쓰는 검색엔진은 구글이므로 구글에서 검색가능하도록 만드는 방법에 대해 알아보자.

## 1. sitemap.xml 생성

sitemap은 웹 크롤링 로봇이 검색엔진에 크롤링하여 URL을 연결시킬 수 있도록 만드는 역할을 한다.

sitemap을 만드는 방법은 크게 두 가지다.

1. Ruby의 gem을 통해 생성하는 방법

ruby command console에 `sudo gem install jekyll-sitemap`을 통해 플러그인을 설치한다.

_config.yml 파일에 아래의 내용을 추가한다.

```yml
plugins:
	- jekyll-sitemap
```

Gemfile 파일에 아래의 내용을 추가한다.

```
gem 'jekyll-sitemap'
```

자세한 내용은 [참고 사이트](<https://github.com/jekyll/jekyll-sitemap>)를 참고한다.

2. 수동으로 sitemap.xml을 생성하는 방법

내 블로그 최상위 디렉토리(_config.yml 파일이 있는 디렉토리라고 생각하자.)에 sitemap.xml 파일을 생성한다.

아래의 내용을 sitemap.xml에 복사한다.

```
---
layout: null
---
{% raw %}
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.sitemaps.org/schemas/sitemap/0.9 http://www.sitemaps.org/schemas/sitemap/0.9/sitemap.xsd" xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  {% for post in site.posts %}
    <url>
      <loc>{{ site.url }}{{ post.url }}</loc>
      {% if post.lastmod == null %}
        <lastmod>{{ post.date | date_to_xmlschema }}</lastmod>
      {% else %}
        <lastmod>{{ post.lastmod | date_to_xmlschema }}</lastmod>
      {% endif %}

      {% if post.sitemap.changefreq == null %}
        <changefreq>weekly</changefreq>
      {% else %}
        <changefreq>{{ post.sitemap.changefreq }}</changefreq>
      {% endif %}

      {% if post.sitemap.priority == null %}
          <priority>0.5</priority>
      {% else %}
        <priority>{{ post.sitemap.priority }}</priority>
      {% endif %}

    </url>
  {% endfor %}
</urlset>
{% endraw %}
```

파일을 저장한 후 jekyll을 실행시켜 블로그에서 확인한다.(127.0.0.1:4000/sitemap.xml)

![jekyll_sitemap](/assets/img/jekyll_sitemap.jpg)

이제 포스트를 작성할 때 sitemap 옵션을 추가하여 포스트 변경 주기, 우선순위 등을 설정할 수 있다.

```
---
layout: post
title: "POST TITLE"
author:
categories:
sitemap:
  changefreq: daily
  priority: 1.0
---
```

여기서 changefreq가 너무 짧으면 빈번한 접속으로 인해 좋지 않은 영향이 있을 수 있다고 하니 하루나 일주일 주기로 하는 것이 좋다.



## 2. robots.txt 생성

robots.txt 파일에 sitemap.xml 파일의 위치를 등록해 놓으면 검색엔진의 크롤러들이 홈페이지를 크롤링하는데 도움을 준다.

sitemap.xml 파일과 같은 디렉토리에 robots.txt 파일을 생성하고 아래와 같이 내용을 입력한다.

```
User-agent: *
Allow: /

Sitemap: "github blog 주소"/sitemap.xml
```

github blog 주소는 내 블로그의 경우 `https://hoik92.github.io/sitemap.xml`처럼 쓰면 되겠다.



## 3. 검색엔진 등록

[Google Search Console](<https://www.google.com/webmasters/#?modal_active=none>) 사이트에 접속한다.

![google_search_console](/assets/img/jekyll_google_search_console.jpg)

SEARCH CONSOLE 버튼을 누른다.

URL 접두어 탭을 선택한 뒤 URL 입력창에 블로그 주소를 입력하고 계속 버튼을 누른다.

![google_search_console_url](/assets/img/jekyll_google_search_console_url.jpg)

소유권 확인 팝업이 뜨면 안내해주는 방법대로 수행한 후 확인 버튼을 눌러 소유권을 확인한다.

권장 확인 방법은 HTML 파일을 블로그에 업로드 하는 방법이지만 나는 이 방법으로 해결이 되지 않아서 HTML 태그 방법을 수행했다.

내가 사용한 Centrarium 테마를 기준으로 설명하자면,

_include 폴더 안에 head.html 파일을 열고 head 태그 안에 주어진 메타 태그를 복사하여 입력하고 저장한 후 git add, git commit, git push를 수행한다.

이후에 소유권 확인 팝업창의 확인 버튼을 누르면 소유권이 확인된다.

![google_search_console_verification](/assets/img/jekyll_google_search_console_verification.jpg)

소유권이 확인되면 왼쪽 메뉴의 Sitemaps 탭을 클릭하여 사이트맵을 추가한다.

![google_search_console_sitemap](/assets/img/jekyll_google_search_console_sitemap.jpg)

제출이 완료되면 사이트맵이 등록된 것을 확인할 수 있다.



하지만 구글에서 검색이 되기까지는 어느 정도 시간이 걸리기 때문에 일주일 정도 후에 확인해보도록 하자.