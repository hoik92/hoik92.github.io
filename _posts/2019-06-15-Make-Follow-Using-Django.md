---
layout: post
title: "[Django] Project01 - Django의 AbstractUser를 상속받아 팔로우 기능 더하기"
author: Hoik Jang
categories: Django
comments: true
sitemap:
  changefreq: daily
  priority: 1.0
---

이제 어느 정도 유저에 관한 내용들을 구성해봤다. 그런데 인스타그램 클론 코딩이라고 하기에는 중요한 한 가지가 빠져있는데, 바로 팔로우 기능이다.
팔로우를 한 유저를 중심으로 그 유저가 포스팅한 게시물을 볼 수 있도록 만드는 것이 포인트인데, 그 시작인 팔로우 기능을 만들어볼 것이다.

이전 포스팅에서는 Profile 모델을 User 모델(settings.AUTH_USER_MODEL)에 1:1 매칭시켜 User 모델에서 Profile 모델을 접근할 수 있도록 구현했다. 이렇게 하면

```python
user = User.objects.get(id=1)
user.profile.nickname
user.profile.image
user.profile.description
```

등과 같이 Profile 모델 필드에 접근할 수 있다.

이번에는 Django 내부에 구현되어 있는 AbstractUser 모델을 상속받아 User 모델을 재정의함으로써 팔로우 기능을 더해볼 것이다. AbstractUser 모델에는 `username`, `first_name`, `last_name`, `email` 등등의 필드가 이미 구현되어 있으며 여기에 팔로우 기능을 할 필드를 추가할 것이다.

AbstractUser 모델에 대한 정보는 [Django 공식 문서](<https://docs.djangoproject.com/en/1.8/_modules/django/contrib/auth/models/>)를 참고하자.

기본적으로 AbstractUser 모델을 상속받아 재정의하는 것을 추천하지는 않는다. 하지만 이러한 방법도 있다는 것을 알아두자는 생각으로 구현해볼 것이다.



## 1. models.py 구성

accounts 앱 내의 models.py에 AbstractUser 모델을 상속받아 MyUser라는 모델을 구현한다.

```python
# models.py
...
from django.contrib.auth.models import AbstractUser

class MyUser(AbstractUser):
    followings = models.ManyToManyField(settings.AUTH_USER_MODEL, related_name="followers")
```

`models.ManyToManyField`를 이용하여 MyUser와 MyUser를 M:N의 관계로 구성한다.

`followings`는 로그인 한 유저가 팔로우하는 유저를 가리키고, `followers`는 한 유저를 팔로우하는 모든 유저를 가리킬 것이다.



## 2. settings.py 수정

기본적으로 `settings.AUTH_USER_MODEL`은 `django.contrib.auth.models`내의 User 모델을 가리키고 있다. 그런데 MyUser라는 새로운 모델이 이를 대신할 것이기 때문에 settings.py에 `settings.AUTH_USER_MODEL`이 가리키는 모델을 다시 지정해준다.

```python
# settings.py
...

AUTH_USER_MODEL = "accounts.MyUser"
```

이제 새로운 모델을 생성했기 때문에 마이그레이션을 해야하는데 기존의 User를 MyUser 모델로 대체하는 것이기 때문에 원래 User 모델로 생성되어있던 계정 정보를 모두 삭제하는 것이 좋다. 쉽게 말해 `db.sqlite3` 파일을 삭제하고 마이그레이션을 하자.



## 3. urls.py 추가

accounts 앱 내의 urls.py에 팔로우 기능을 할 url을 뚫어준다.

```python
# urls.py
...

urlpatterns = [
    ...
    path('<str:username>/follow/', views.follow, name="follow"),
]
```



## 4. views.py 팔로우 기능 구현

accounts 앱 내의 views.py에 팔로우 기능을 구현한다.

```python
# views.py
...
from .models import MyUser
from django.contrib.auth.decorators import login_required

...

@login_required
def follow(request, username):
    user = request.user
    follow_user = get_object_or_404(MyUser, username=username)
    if user in follow_user.followers.all():
        user.followings.remove(follow_user)
        # follow_user.followers.remove(user)
    else:
        user.followings.add(follow_user)
        # follow_user.followers.add(user)
    return redirect('people', username)
```

팔로우는 로그인을 한 경우에만 가능하기 때문에 `login_required`라는 데코레이터를 이용한다.

로그인한 유저를 `user`, 팔로우 대상 유저를 `follow_user`로 지정한다.

조건문을 이용하여 두 가지의 경우로 나눈다.

1. 팔로우할 대상 유저를 이미 팔로우하고 있는 경우

팔로우를 취소한다.

2. 팔로우할 대상 유저를 아직 팔로우하지 않은 경우

대상 유저를 팔로우한다.



## 4. people.html 팔로우 버튼 추가

people.html 파일을 수정하여 팔로우 버튼을 추가한다.

```html
<!-- accounts/templates/accounts/people.html -->
...
{% raw %}
{% if user.is_authenticated %}
	{% if person.username == user.username %}
	<a href="{% url 'accounts:profile' %}">
        <button class="btn btn-success">프로필 편집</button>
	</a>
	{% else %}
		{% if user in person.followers.all %}
		<a href="{% url 'accounts:follow' person.username %}">
            <button class="btn btn-outline-info">언팔로우</button>
		</a>
		{% else %}
		<a href="{% url 'accounts:follow' person.username %}">
            <button class="btn btn-info">팔로우</button>
		</a>
		{% endif %}
	{% endif %}
{% endif %}{% endraw %}
```

위의 코드를 템플릿 내의 적절한 위치에 추가하여 팔로우 버튼을 추가한다.

![django_follow1](/assets/img/django/django_follow1.jpg)

![django_follow2](/assets/img/django/django_follow2.jpg)