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
	'accounts', # 웬만하면 콤마(,) 찍어주자. 다음에 앱을 추가할 때 빼먹는 경우가 많다.
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
# view나 template에서 해당 이름을 이용해 url에 요청을 보낼 수 있다.

urlpatterns = [
    path('signup/', views.signup, name="signup"),
    # "<내 URL>/accounts/signup/" 주소로 요청이 오면 views.py 안에 signup이라는 함수를 찾아 실행한다.
]
```



## 3. views.py 수정

accounts 폴더 안에 views.py를 열고 다음을 입력한다.

```python
# views.py
from django.shortcuts import render, redirect
from django.contrib.auth import login as user_login
from django.contrib.auth.forms import UserCreationForm
# Django 프레임워크가 구현해 놓은 회원가입 폼을 import 한다.

def signup(request): # urls.py에 views.signup이 이 함수를 가리킨다.
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid(): # 회원가입 폼에서 적어 보낸 요청이 유효한지 검사한다.
            user = form.save() # 유효한 내용이면 이 회원 정보를 데이터베이스에 저장한다. 그 유저 정보를 리턴한다.
            user_login(request, user) # 유저 정보를 이용해 로그인한다.
        return redirect('accounts:signup')
    	# redirect 시 urls.py의 <app_name>:<name>으로 요청을 보낸다.
    else:
        form = UserCreationForm() # 비어있는 회원가입 폼을 생성한다.
        return render(request, 'accounts/form.html', {'form': form})
    	# forms.html 파일을 렌더한다. 이때 위에서 생성한 회원가입 폼을 'form'이라는 이름으로 함께 보낸다.(딕셔너리)
```



## 4. template 생성

accounts 폴더 안에 templates 폴더를 생성한 후 그 안에 accounts 폴더를 생성한다.

accounts/templates/accounts 폴더 안에 form.html 파일을 생성하고 다음과 같이 입력한다.

```html
{% raw %}
<h1>
    <!-- DTL(Django Template Language)로 중괄호와 '%'기호를 이용하여 마치 Python과 비슷하게 조건과 반복문을 사용할 수 있다. -->
    <!-- 만약 요청한 url에 'signup'이라는 단어가 들어있으면 -->
	{% if "signup" in request.path %}
	회원가입
	{% else %}
	로그인
	{% endif %}
    <!-- 위의 조건을 통해 하나의 템플릿 파일로 회원가입과 로그인 페이지를 구성할 수 있다. -->
</h1>

<form method="POST">
	{% csrf_token %}
	{{ form }}
    <!-- views.py에서 딕셔너리 형태로 넘겨받은 form을 중괄호 두개를 이용하여 사용할 수 있다. -->
	<button type="submit">가입</button>
</form>
{% endraw %}
```

위와 같이 파일들을 생성하고 수정하면 아래와 같은 화면을 볼 수 있다.

![django_signup](/assets/img/django/django_signup.jpg)

위의 화면을 보면 url에 `signup`이라는 단어가 있으므로 `h1` 태그로 '회원가입'이라 표시된다.

사용자 이름, 비밀번호 등의 입력 폼은 Django 프레임워크에서 구현되어있고 이를 편집하여 사용할 수도 있다.