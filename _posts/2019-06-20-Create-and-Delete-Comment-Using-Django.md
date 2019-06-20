---
layout: post
title: "[Django] Comment 모델을 생성하여 댓글 기능 추가"
author: Hoik Jang
categories: Django
comments: true
sitemap:
  changefreq: daily
  priority: 1.0
---

Post 모델을 구성하여 게시물을 작성하고, 수정하고, 삭제하는 기능을 구현해봤다.

이제 이 게시물에 댓글을 달 수 있게 만들어볼 것이다.



## 1. models.py에 Comment 모델 구현

posts 앱 내의 models.py에 Comment 모델을 구현할 것이다.

```python
# models.py
...

class Comment(models.Model):
    content = models.CharField(max_length=200)
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    post = models.ForeignKey(Post, on_delete=models.CASCADE, related_name="comments")
    
    def __str__(self):
        return f"{self.id}: {self.content} - {self.user.username}"
```

content 컬럼에는 작성한 댓글, user 컬럼에는 댓글을 작성한 유저, post 컬럼에는 해당 댓글이 작성된 게시물이 저장될 것이다.

이후에 마이그레이션을 수행한다.

```shell
python manage.py makemigrations
python manage.py migrate
```





## 2. forms.py 댓글 폼 구성

posts 앱 내의 forms.py에 댓글을 작성할 수 있는 폼을 구성한다.

```python
# forms.py
...
from .models import Comment

...
class CommentForm(forms.ModelForm):
    class Meta:
        model = Comment
        fields = ['content',]
```



## 3. urls.py 경로 추가

posts 앱 내의 urls.py에 댓글을 작성하고 삭제하는데 필요한 url을 구성한다.

```python
# urls.py
...

urlpatterns = [
    ...
    path('<int:post_id>/create/comment/', views.create_comment, name="create_comment"),
    path('<int:post_id>/delete/comment/<int:comment_id>/', views.delete_comment, name="delete_comment"),
]
```

지금은 RESTful과 매우 거리가 멀지만 일단 넘어가자..

다음에 Javascript도 사용하여 프로젝트를 진행할 때 조금 더 깔끔하게 해보자.



## 4. views.py 액션 구성

posts 앱 내의 views.py에 댓글 추가와 삭제 액션을 구성한다.

```python
# views.py
...
from .models import Comment
from .forms import CommentForm

def index(request):
    def index(request):
    posts = Post.objects.all()
    form = CommentForm()
    return render(request, 'posts/index.html', {'posts': posts, 'form': form})

...

@require_POST
@login_required
def create_comment(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    form = CommentForm(request.POST)
    if form.is_valid():
        comment = form.save(commit=False)
        comment.user = request.user
        comment.post = post
    return redirect('posts:index')

@login_required
def delete_comment(request, post_id, comment_id):
    comment = get_object_or_404(Comment, id=comment_id)
    if request.user == comment.user:
        comment.delete()
    return redirect('posts:index')
```



## 5. template 수정

기본적으로 게시물들이 index.html에서 보이게 템플릿을 구성했으므로 댓글을 작성하는 폼도 index.html에 보이도록 템플릿을 수정한다.

```html
<!-- posts/templates/posts/index.html -->{% raw %}
...
{% for post in posts %}
	...

    <!-- 댓글 입력 form -->
    {% if user.is_authenticated %}
    <form action="{% url 'posts:create_comment' post.id %}" method="POST">
        {% csrf_token %}
        {% bootstrap_form form %}
        <button type="submit">등록</button>
    </form>
    {% endif %}

    <!-- 댓글 출력 -->
    {% if post.comments.all %}
    <ul>
        {% for comment in post.comments.all %}
        <li>
            <strong>{{ comment.user.username }}</strong>
            <p>{{ comment.content }}
                {% if user == comment.user %}
                <a href="{% url 'posts:delete_comment' post.id comment.id %}">
                    [삭제]
                </a>
                {% endif %}
            </p>
        </li>
        {% endfor %}
    </ul>
    {% endif %}

	...
{% endfor %}{% endraw %}
```

위의 코드를 참고하여 적절하게 꾸미자.

![django_comment](/assets/img/django/django_comment.jpg)