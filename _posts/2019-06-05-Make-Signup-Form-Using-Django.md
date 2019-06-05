---
layout: post
title: "[Django] Django를 이용하여 회원가입 페이지 만들기"
author: Hoik Jang
categories: Django
comments: true
sitemap:
  changefreq: daily
  priority: 1.0
---

Django를 이용해서 웹 페이지를 만들어볼 것이다.

Django 내부에는 기본적인 구조, 필요한 코드 등이 구현되어 있기 때문에 우리는 이를 잘 활용하여 웹 어플리케이션을 제작하게 된다.

Django는 MTV(Model-Template-View) 패턴으로 이루어진다.

![django_mtv](https://wayhome25.github.io/assets/post-img/django/mtv.png)

출처: [초보몽키](<https://wayhome25.github.io/django/2017/02/28/django-03-lotto-project-1/>)님 블로그

우리가 브라우저를 통해 URL을 입력하여 서버에 요청을 보내면 Django 프로젝트 폴더 내의 urls.py에서 URL을 파싱한 후 View에 전달한다.

View는 Model로부터 데이터를 받아 처리한 후 Template 형태로 응답을 보낸다.

Template은 유저가 실제로 보는 화면을 담당한다.(html, ..)

많이 건너뛰는 느낌이 있지만 나의 복습을 위해 회원가입 페이지부터 만들어보자.

## 1. 앱 생성 및 환경 설정

shell에 다음을 입력하여 앱을 생성한다.

```shell
python manage.py startapp accounts
```

그러면 프로젝트 폴더 안에 accounts라는 이름을 가진 앱 폴더가 생성된다.

그 다음 settings.py에 해당 앱을 등록하고 수정한다.

```python
# settings.py
INSTALLED_APPS = [
	...
	'accounts', # 웬만하면 콤마(,) 찍어주자.
]

LANGUAGE_CODE = 'ko-kr'
TIME_ZONE = 'Asia/Seoul'
```



## 2. urls.py 수정

instagram 폴더 안의 urls.py를 열고 수정한다.

```python
# urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/', include('accounts.urls')),
    # "<내 URL>/accounts/" 주소로 요청이 오면 accounts 앱의 urls.py로 요청을 넘긴다.
]
```

accounts 폴더 안에 urls.py를 생성하고 다음을 입력한다.

```python
from django.urls import path
from . import views

app_name = "accounts"

urlpatterns = [
    path('signup/', views.signup, name="signup"),
    # "<내 URL>/accounts/signup/" 주소로 요청이 오면 views.py 안에 signup이라는 함수를 찾아 실행한다.
]
```

