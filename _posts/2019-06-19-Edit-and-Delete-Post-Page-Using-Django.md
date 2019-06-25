---
layout: post
title: "[Django] Project01 - 포스트 수정 및 삭제하기"
author: Hoik Jang
categories: Django
comments: true
sitemap:
  changefreq: daily
  priority: 1.0
---

이번에는 작성한 포스트를 수정하고 삭제하는 기능을 구현해볼 것이다.

우리가 만들었던 Post 모델은 아래와 같다.

```python
class Post(models.Model):
    content = models.CharField(max_length=300, blank=True)
    image = models.ImageField(blank=True, upload_to='posts')
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name="posts")
    like_users = models.ManyToManyField(settings.AUTH_USER_MODEL, related_name="like_posts", blank=True)
```

네 개의 필드 중 우리가 수정할 필요가 있는 부분은 content와 image 이다. 나머지는 포스트를 작성할 때, 혹은 시간이 지나면서 누군가에 의해 이미 정해진 것이므로 이를 수정할 필요는 없다.

따라서 포스트 작성 시 사용하기 위해 구성했던 PostForm을 그대로 이용하여 포스트를 수정할 수 있다.



## 1. urls.py 경로 추가

posts 앱 내의 urls.py에 수정, 삭제 요청을 받을 url 경로를 추가한다.

```python
# urls.py
...

urlpatterns = [
    ...
    path('edit/<int:post_id>/', views.edit_post, name="edit_post"),
    path('delete/<int:post_id>/', views.delete_post, name="delete_post"),
]
```



## 2. views.py 액션 구성

posts 앱 내의 views.py에 포스트를 수정, 삭제하는 액션을 구성한다.

```python
# views.py
...
from django.views.decorators.http import require_POST

@login_required
def edit_post(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    if post.user == request.user:
        if request.method == 'POST':
            form = PostForm(request.POST, request.FILES, instance=post)
            if form.is_valid():
                form.save()
                return redirect('posts:index')
            return redirect('posts:edit_post')
        else:
            form = PostForm(instance=post)
            return render(request, 'posts/form.html', {'form': form})
    return redirect('posts:index')

@login_required
@require_POST
def delete_post(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    if post.user == request.user:
        post.delete()
    return redirect('posts:index')
```

`instance=post`로 PostForm이 가리키는 데이터가 어떤 것인지 지정해줌으로써 그 데이터를 수정할 수 있도록 만든다.

템플릿에 현재 로그인한 유저가 작성한 포스트에만 수정, 삭제할 수 있는 버튼이 보이도록 만들 것이지만 `if post.user == request.user` 조건문을 이용함으로써 잘못된 경로로 요청이 오는 것을 한 번 더 막고자 했다.

`require_POST` 데코레이터를 사용하여 POST 요청으로만 삭제가 가능하도록 구현했다.



## 3. index.html 템플릿 수정

index.html 템플릿에 수정, 삭제 버튼을 추가한다.

```html
<!-- index.html -->{% raw %}
<a href="{% url 'posts:edit_post' post.id %}"><button>수정</button></a>
<form action="{% url 'posts:delete_post' post.id %}" method="POST">
    {% csrf_token %}
    <button type="submit">삭제</button>
</form>{% endraw %}
```

위의 버튼을 적절한 위치에 추가한다.

삭제 버튼의 경우 버튼 한 번 잘못 눌러서 포스트가 삭제될 수 있으므로 모달 등의 컴포넌트를 이용해도 좋다.