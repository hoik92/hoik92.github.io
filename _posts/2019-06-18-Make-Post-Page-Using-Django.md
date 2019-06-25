---
layout: post
title: "[Django] Project01 - Post 모델을 생성하여 포스트 작성하기"
author: Hoik Jang
categories: Django
comments: true
sitemap:
  changefreq: daily
  priority: 1.0
---

이전 포스팅들을 통해 회원가입부터 유저를 팔로우하는 기능까지 만들어보았다.

이제는 유저에 관련된 내용이 아니라 유저들이 작성하여 업로드할 포스트에 대한 내용을 모델링할 것이다.



## 1. posts 앱 생성

유저 정보에 관한 앱인 accounts 앱을 만들었던 것처럼 포스팅에 관한 앱인 posts 앱을 생성한다.

보통 앱의 이름은 복수형으로 짓는 것이 일반적이다.

```shell
python manage.py startapp posts
```



## 2. settings.py 앱 추가

settings.py의 INSTALLED_APPS에 방금 생성한 posts 앱을 추가한다.

```python
# settings.py
...

INSTALLED_APPS = [
    ...
    'posts',
]
```



## 3. urls.py 경로 추가

instagram 프로젝트 폴더 내의 urls.py에 경로를 추가한다.

```python
# urls.py
...

urlpatterns = [
    ...
    path('posts/', include('posts.urls')),
]
```

이렇게 하면 `<BASE_URL>/posts/` url로 요청이 오면 posts 앱 내의 urls.py에 작성된 경로를 통해 액션이 수행된다.



## 4. Post 모델 구성

posts 앱 내의 models.py에 Post 모델을 구성한다.

```python
# models.py
from django.db import models
from django.conf import settings

class Post(models.Model):
    content = models.CharField(max_length=300, blank=True)
    image = models.ImageField(blank=True, upload_to='posts')
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name="posts")
    like_users = models.ManyToManyField(settings.AUTH_USER_MODEL, related_name="like_posts", blank=True)
    
    def __str__(self):
        return f"{self.id}: {self.content}"
```

여기서 content 필드는 작성자가 쓴 글이 저장될 필드, image 필드는 작성자가 업로드한 이미지의 url이 저장될 필드이다.

user 필드는 해당 포스트를 작성한 작성자에 대한 정보가 들어갈 필드이며, 하나의 포스트에 대한 작성자는 한 명이기 때문에 1:N 매칭을 이루어주는 ForeignKey 필드를 이용했다.

like_users 필드는 해당 포스트에 대한 좋아요 기능을 구현하기 위한 필드이다. 하나의 포스트에는 여러 사람이 좋아요를 누를 수 있고, 한 명의 유저는 여러 개의 포스트에 좋아요를 누를 수 있으므로 M:N 매칭을 이루는 ManyToManyField를 이용했다.



* 참고

만약 업로드된 포스트들을 삭제하고 싶을 때, 위와 같은 Post 모델의 경우 그 내용물을 쉽게 삭제할 수 있다. 하지만 이는 데이터베이스에 저장된 데이터를 삭제하는 것일 뿐이다. 즉, 업로드된 이미지 파일이 실제로 삭제되는 것이 아니다. 따라서 포스트 데이터를 삭제할 때 관련된 이미지 파일도 같이 삭제하기 위해서는 models.py에 다음 내용을 추가하면 된다.

```python
# models.py
...
from django.db.models.signals import post_delete
from django.dispatch import receiver

class Post(models.Model):
    ...
    

@receiver(post_delete, sender=Post)
def submission_delete(sender, instance, **kwargs):
    instance.image.delete(False)
```



모델 구성을 완료한 후 마이그레이션하여 DB에 반영한다.

```shell
python manage.py makemigrations
python manage.py migrate
```



## 5. forms.py 구성

posts 앱 내에 forms.py 파일을 생성하여 포스트를 작성할 때 사용할 폼을 구성한다.

```python
# forms.py
from django import forms
from .models import Post

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['content', 'image',]
```

Post 모델의 필드는 content, image, user, like_users 네 가지이다. 그 중에 폼을 구성하는 요소로 두 가지만 사용한 이유가 있다.

user 필드의 경우 로그인을 한 유저가 글을 작성하는 것이기 때문에 로그인한 유저의 정보를 바로 넣을 것이다.(포스트를 작성한 사람이 누구인지를 작성자가 선택하게 한다면 매우 번거로울 것이다.)

like_users 필드의 경우 이미 작성된 포스트들을 보면서 좋아요 버튼을 누를 경우에 변경될 내용이기 때문에 포스트를 작성하는 단계에는 작성할 필요가 없다.



## 6. urls.py 경로 설정

posts 앱 내에 urls.py 파일을 생성한 후 경로를 추가한다.

```python
# urls.py
from django.urls import path
from . import views

app_name = "posts"

urlpatterns = [
    path('index/', views.index, name="index"),
    path('create/', views.create_post, name="create_post"),
]
```

index 페이지는 모든 포스트를 볼 수 있게 만들 것이고 create 페이지는 포스트를 작성하는 페이지로 구성할 것이다.



## 7. views.py 액션 구성

posts 앱 내의 views.py에 url 요청이 들어올 경우 동작할 액션을 정의한다.

```python
# views.py
from django.shortcuts import render, redirect, get_object_or_404
from .forms import PostForm
from .models import Post

# Create your views here.
def index(request):
    posts = Post.objects.all()
    # Post 모델을 기반으로 생성된 테이블의 모든 데이터를 가져온다.
    return render(request, 'posts/index.html', {'posts': posts})

def create_post(request):
    if request.method == 'POST':
        form = PostForm(request.POST, request.FILES)
        if form.is_valid():
            post = form.save(commit=False)
            # commit=False일 경우 실제 DB에는 적용하지 않는다.
            # 아직 작성자 정보가 담겨있지 않기 때문에 실제 DB에 적용하는 것을 지연시키고,
            # 작성자 정보를 넣어준 후 DB에 적용한다.
            post.user = request.user
            post.save()
            return redirect('posts:index')
        return redirect('posts:create_post')
    else:
        form = PostForm()
        title = '새 글'
        return render(request, 'posts/form.html', {'form': form, 'title': title})
```



## 8. 템플릿 구성

posts 앱 내에 templates 폴더를 생성하고 그 안에 posts 폴더를 생성한 후 템플릿을 구성한다.

```html
<!-- index.html -->{% raw %}
{% extends 'base.html' %}

{% block body %}
<div class="card-columns">
    {% for post in posts %}
    <div class="card">
        {% if post.image %}
        <img src="{{ post.image.url }}" class="card-img-top" alt="...">
        {% endif %}
        <div class="card-body">
        <p class="card-text">{{ post.content }}</p>
        </div>
    </div>
    {% endfor %}
</div>
{% endblock %}{% endraw %}
```

```html
<!-- form.html -->{% raw %}
{% extends 'base.html' %}
{% load bootstrap4 %}


{% block body %}
<h1 class="text-center">{{ title }}</h1>
<form method="POST" enctype="multipart/form-data">
    {% csrf_token %}
    {% bootstrap_form form %}
    <button class="btn btn-primary">완료</button>
</form>
{% endblock %}{% endraw %}
```

![django_create_post](/assets/img/django/django_create_post.jpg)

![django_index](/assets/img/django/django_index.jpg)