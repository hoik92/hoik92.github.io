---
layout: post
title: "[Django] Django의 AbstractUser를 상속받아 팔로우 기능 더하기"
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