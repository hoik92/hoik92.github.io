---
layout: post
title: "[Django] Project01 - Django를 이용하여 프로필 페이지 만들기"
author: Hoik Jang
categories: Django
comments: true
sitemap:
  changefreq: daily
  priority: 1.0
---

Django 내부에 구현된 User 모델은 기본적으로 `username`, `password`, `email` 등 여러 정보를 담을 수 있는 필드가 구성되어 있다.

이번에는 여기에 더하여 Profile 모델을 만들어 User 모델과 1:1로 연결시켜 프로필 사진, 별명 등을 추가할 수 있도록 만들 것이다.

그리고 이를 간단히 볼 수 있는 프로필 페이지를 구현해 볼 것이다.

## 1. models.py 모델 생성

accounts 앱 내의 models.py에 Profile 모델을 생성한다.

```python
# models.py
from django.db import models
from django.conf import settings

class Profile(models.Model):
    user = models.OneToOneField(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    # User모델과 Profile을 1:1로 연결
    description = models.TextField(blank=True)
    nickname = models.CharField(max_length=40, blank=True)
    image = models.ImageField(blank=True)
```

여기서 이미지를 저장하는 필드인 `models.ImageField`를 사용하기 위해 Pillow 패키지를 설치한다.

```shell
pip install pillow
```

Pillow 패키지는 Python을 이용하여 이미지를 처리하고 싶을 때 사용되는 패키지이다.

models.py에 모델을 설계했으면 이 모델을 생성하여 데이터베이스에 적용시켜야 한다.

```shell
python manage.py makemigrations
python manage.py migrate
```



## 2. settings.py 미디어 저장 경로 지정

Django에서는 정적 파일을 static file과 media file로 구분한다.

static file은 웹 서비스에 제공하기 위해 준비한 정적 파일이고, media file은 웹 서비스 사용자가 서버에 업로드하는 정적 파일이다.

지금 단계에서는 사용자가 자신의 프로필 사진을 업로드할 수 있도록 만들 것이기 때문에 이는 media file에 해당한다.

media file을 저장하기 위해서는 이 파일이 어떤 url을 타고 들어와 어디에 모일지 지정해주어야 한다. 이를 settings.py와 urls.py에서 설정한다.

이미지 등의 미디어 파일을 저장할 경로를 settings.py에 지정한다.

```python
# settings.py
...

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, MEDIA_URL)
```

이렇게 MEDIA_ROOT를 지정하면 미디어 파일이 저장되는 경로가 `<BASE_DIR>/media/`가 된다.

그리고 파일이 타고 들어올 url을 instagram 프로젝트 폴더 내의 urls.py에 지정한다.

```python
# urls.py
...
from django.conf import settings
from django.conf.urls.static import static

...

urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```



## 3. urls.py 

instagram 프로젝트 폴더 내의 urls.py에 프로필을 페이지의 url을 등록한다.

```python
# urls.py
...
from accounts import views as accounts_views

urlpatterns = [
    ...
	path('<str:username>/', accounts_views.people, name="people"),
]
```

`<BASE_URL>/<username>`의 형태로 url 요청이 들어오면 `<username>`을 문자열로 변환하여 accounts 앱 내의 views.py에 구현된 people 메소드에 인자로 넘겨준다.



## 4. views.py 액션 구성

프로필 페이지를 로드하는 url로 요청이 들어오면 수행될 메소드를 구현한다.

```python
# views.py
from django.shortcuts import get_object_or_404
...
from .models import Profile

...

def people(request, username): # urls.py에서 넘겨준 인자를 username으로 받는다.
	person = get_object_or_404(get_user_model(), username=username)
    return render(request, 'accounts/people.html', {'person': person})
```



## 5. people.html 구성

프로필 페이지 템플릿을 구성한다.

```html
<!-- accounts/templates/accounts/people.html -->{% raw %}
{% extends 'base.html' %}

{% block body %}
<div class="row">
    <div class="col-4">
        <img src={% if person.profile.image %}"{{ person.profile.image.url }}"{% else %}"https://i.stack.imgur.com/34AD2.jpg"{% endif %} 
            width="150rem" style="border-radius: 50%;" alt="">
    </div>
    <div class="col-8">
        <h2>{{ person.username }}
            {% if person.username == user.username %}
            <a href="{% url 'accounts:profile' %}"><button class="btn btn-success">프로필 편집</button></a>
            {% endif %}
            <!-- 다음 포스트에 프로필 편집 페이지를 구성할 것이다. -->
        </h2>
        <p>{{ person.profile.nickname }}</p>
        <p>{{ person.profile.description }}</p>
    </div>
</div>
{% endblock %}{% endraw %}
```

멋은 없지만 어쨌든 프로필 페이지를 완성했다.

이 프로젝트를 진행하면서 프로필 수정 페이지도 구성하고, 포스트를 업로드하는 기능 등을 구현하고 나면, 이 프로필 페이지도 조금 더 수정할 것이다.