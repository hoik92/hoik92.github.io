---
layout: post
title: "[Django] Django를 이용하여 프로필 수정 페이지 만들기"
author: Hoik Jang
categories: Django
comments: true
sitemap:
  changefreq: daily
  priority: 1.0
---

앞선 포스팅에서 Profile 모델을 구성하여 User 모델과 1:1 매칭시키고 이를 간단하게 볼 수 있는 프로필 페이지를 만들었다.

Django 내부에 구현된 UserCreationForm 클래스는 회원가입, AuthenticationForm 클래스는 로그인을 할 때 사용할 수 있다.

여기에 추가적으로 UserChangeForm 클래스를 이용하여 유저 정보를 변경할 수 있는 프로필 수정 페이지를 만들어볼 것이다.

앞서 Profile 모델을 다음과 같이 설계했다.

```python
# models.py
class Profile(models.Model):
    user = models.OneToOneField(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    # User모델과 Profile을 1:1로 연결
    description = models.TextField(blank=True)
    nickname = models.CharField(max_length=40, blank=True)
    image = models.ImageField(blank=True)
```

이 모델을 이용한 수정 폼을 구현할 것이다.

## 1. urls.py

accounts 앱 내부의 urls.py에 프로필 수정 페이지 요청을 받을 url을 구성한다.

```python
# urls.py
...

urlpatterns = [
	...
	path('profile/', views.profile, name="profile"),
]
```



## 2. forms.py 폼 구성

Django 내부에는 사용자 정보를 편집할 수 있는 UserChangeForm이 구현되어 있다. 하지만 이 폼은 사용자의 모든 정보를 수정할 수 있도록 구성되어 있기 때문에(심지어 권한도..) 이를 상속받아 새로운 폼을 구성할 것이다.

그리고 추가로 위에서 만든 Profile 모델의 내용을 편집할 수 있는 폼도 만들 것이다.

accounts 앱 내의 forms.py에 내용을 추가한다.

```python
# forms.py
...
from django.contrib.auth.forms import UserChangeForm
from .models import Profile

class CustomUserChangeForm(UserChangeForm):
	password = None
    # UserChangeForm에서는 password를 수정할 수 없다.
    # 하지만 이렇게 None 값으로 지정해주지 않으면 password를 변경할 수 없다는 설명이 화면에 표현된다.
    class Meta:
        model = get_user_model()
        fields = ['email', 'first_name', 'last_name',]
        
class ProfileForm(models.Model):
    nickname = forms.CharField(label="별명", required=False)
    description = forms.CharField(label="자기소개", required=False, widget=forms.Textarea())
    image = forms.ImageField(label="이미지", required=False)
   	# 위의 내용을 정의하지 않아도 상관없지만, 화면에 출력될 때 label이 영문으로 출력되는 것이 싫어서 수정한 것이다..
    class Meta:
        model = Profile
        fields = ['nickname', 'description', 'image',]
```



## 3. views.py 액션 구성

프로필을 변경하는 url로 요청이 들어오면 수행될 메소드를 구현한다.

```python
# views.py
...
from .forms import CustomUserChangeForm, ProfileForm
from .models import Profile

...

def profile(request):
	if request.method == 'POST':
        user_change_form = CustomUserChangeForm(request.POST, instance=request.user)
        profile_form = ProfileForm(request.POST, request.FILES, instance=request.user.profile)
        if user_change_form.is_valid() and profile_form.is_valid():
            user = user_change_form.save()
            profile_form.save()
            return redirect('people', user.username)
        return redirect('accounts:profile')
    else:
        user_change_form = CustomUserChangeForm(instance=request.user)
        # 새롭게 추가하는 것이 아니라 수정하는 것이기 때문에
        # 기존의 정보를 가져오기 위해 instance를 지정해야 한다.
        profile, create = Profile.objects.get_or_create(user=request.user)
        # Profile 모델은 User 모델과 1:1 매칭이 되어있지만
        # User 모델에 새로운 인스턴스가 생성된다고 해서 그에 매칭되는 Profile 인스턴스가 생성되는 것은 아니기 때문에
        # 매칭되는 Profile 인스턴스가 있다면 그것을 가져오고, 아니면 새로 생성하도록 한다.
        profile_form = ProfileForm(instance=profile)
        return render(request, 'accounts/profile.html', {
            'user_change_form': user_change_form,
            'profile_form': profile_form
        })
```



## 4. profile.html 구성

프로필 수정 페이지 템플릿을 구성한다.

```html
<!-- accounts/templates/accounts/profile.html -->{% raw %}
{% extends 'base.html' %}

{% load bootstrap4 %}

{% block body %}
    <h1 class="text-center">회원정보 변경</h1>
    <form method="POST" enctype="multipart/form-data">
    <!-- 파일을 업로드할 때 form method는 POST여야하고 enctype을 multipart/form-data로 설정해야 한다. -->
        {% csrf_token %}
        {% bootstrap_form user_change_form %}
        {% bootstrap_form profile_form %}
        <button type="submit" class="btn btn-primary">수정</button>
    </form>
{% endblock %}{% endraw %}
```

기본적으로 이 프로젝트는 기능 구현에 초점을 둔 것이기 때문에 웹 페이지 디자인에 시간을 거의 투자하지 않았다. 그래서 페이지가 멋이 없다..

![django_profile_edit](/assets/img/django/django_profile_edit.jpg)

![django_profile](/assets/img/django/django_profile.jpg)