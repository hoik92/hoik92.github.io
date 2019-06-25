---
layout: post
title: "[Django] Project01 - Django를 이용하여 로그인, 로그아웃 만들기"
author: Hoik Jang
categories: Django
comments: true
sitemap:
  changefreq: daily
  priority: 1.0
---

회원가입은 만들었는데 로그인과 로그아웃을 할 수가 없다.

그래서 로그인 페이지와 로그아웃 기능을 구현해 볼 것이다.

내가 보통 한 가지 기능을 구현할 때 수행하는 작업 순서는 Model, Url, View, Template이다. 이게 정답은 아니거니와 틀렸을 수도 있지만 기본적으로는 이 순서대로 진행할 것이다.

회원가입을 비롯하여 로그인과 로그아웃에 쓰이는 계정 정보에 대한 모델은 Django에 미리 구현되어 있기 때문에 Model 부분은 생략하는 것이다.

## 1. urls.py 수정

accounts 앱 안의 urls.py를 열고 내용을 추가한다.

```python
...

urlpatterns = [
	...
	path('login/', views.login, name="login"),
    # URL/accounts/login 의 url로 요청이 들어오면 views.py의 login 메소드를 수행한다.
    path('logout/', views.logout, name="logout"),
]
```

## 2. views.py 수정

accounts 앱 안의 views.py를 열고 로그인과 로그아웃 기능을 구현한다.

```python
...
from django.contrib.auth import login as user_login
# 함수명을 login이라 쓸 것이므로 django내에 로그인 기능이 구현된 login 함수를 user_login으로 변경하여 import
from django.contrib.auth import logout as user_logout
from django.contrib.auth.forms import AuthenticationForm
# django내에 구현된 로그인 폼

def login(request):
    if request.method == 'POST':
        form = AuthenticationForm(request, request.POST)
        # 회원가입과 다르게 맨 앞의 인자로 request가 들어간다.
        if form.is_valid():
            user_login(request, form.get_user())
            return redirect('posts:index')
        return redirect('accounts:login')
    else:
        form = AuthenticationForm()
        return render(request, 'accounts/form.html', {'form': form})

def logout(request):
    user_logout(request)
    return redirect('posts:index')
```

위의 코드들은 최소한의 기능만을 구현한 것이기 때문에 추가할 내용이 있으면 개인적으로 더 추가하여 구현하면 된다.