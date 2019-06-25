---
layout: post
title: "[Django] Project01 - 게시물 좋아요 기능 구현"
author: Hoik Jang
categories: Django
comments: true
sitemap:
  changefreq: daily
  priority: 1.0
---

sns의 중요한 기능 중 하나가 좋아요 기능일 것이다.

이미 Post 모델을 구현할 때 이 기능을 염두해 두고 만들었다.

```python
class Post(models.Model):
    content = models.CharField(max_length=300)
    image = models.ImageField(blank=True, upload_to="posts")
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name="posts")
    like_users = models.ManyToManyField(settings.AUTH_USER_MODEL, related_name="like_posts", blank=True)
```

like_users 필드를 이용하여 좋아요 기능을 만들어볼 것이다.



## 1. urls.py 경로 추가

posts 앱 내의 urls.py에 좋아요 기능을 수행할 경로를 추가한다.

```python
# urls.py
...

urlpatterns = [
	...
	path('like/<int:post_id>/', views.like_post, name="like_post"),
]
```



## 2. views.py 좋아요 기능 구현

posts 앱 내의 views.py에 좋아요 기능을 구현한다.

```python
# views.py
...

@login_required
def like_post(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    if request.user in post.like_users.all():
        post.like_users.remove(request.user)
    else:
        post.like_users.add(request.user)
```

사용자가 좋아요 버튼을 누를 경우 해당 사용자가 해당 게시물에 좋아요를 누른 상태면 이를 취소하고, 아니면 추가하는 코드이다.



## 3. template 수정

index.html의 적절한 위치에 좋아요 버튼을 추가한다.

나는 [Font Awesome](<https://fontawesome.com/>) 사이트에서 하트 모양의 아이콘을 사용하여 버튼을 구성했다.

base.html에 Font Awesome CDN을 추가한 후에 아래의 코드를 적절한 위치에 넣어주면 되겠다.

```html
<!-- index.html -->{% raw %}
...

{% for post in posts %}
	...
	
    <!-- 좋아요 버튼 -->
    <a href="{% url 'posts:like_post' post.id %}">
        {% if user in post.like_users.all %}
        <i class="fas fa-heart"></i>
        {% else %}
        <i class="far fa-heart"></i>
        {% endif %}
    </a>

	...
{% endfor %}{% endraw %}
```



추가적으로 자신이 작성한 게시물과 자신이 팔로우한 유저의 게시물만 볼 수 있도록 만들 수도 있다.

```python
# views.py
...
from django.db.models import Q

def index(request):
    posts = Post.objects.filter(
        Q(user=request.user) | Q(user_id__in=request.user.followings.all())
    )
    ...
```

위처럼 Q를 이용하여 ORM으로 쿼리문을 작성할 때 or이나 and 연산을 수행할 수 있다.



이정도로 이번 instagram 따라하기 프로젝트를 마무리 지으려 한다.

일단 겉으로 보기에 많이 부족하고 클린 코드라고 보기에도 좀 민망하지만 이러한 과정들을 거치는 것이 좋은 개발자로 한 걸음 가는 방법이 아닐까 싶다.



다음에는 Vue.js를 조금 활용하는 프로젝트를 해볼까 한다.