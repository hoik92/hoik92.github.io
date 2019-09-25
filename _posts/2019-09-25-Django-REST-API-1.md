---
layout: post
title: "[Django] Project02 - REST API 만들기"
author: Hoik Jang
categories: Django
comments: true
sitemap:
  changefreq: daily
  priority: 1.0
---

REST API를 만들어보자.

REST란 Representational State Transfer의 약자로 자원을 이름으로 구분하여 상태를 주고 받는 것을 의미한다.

내 나름대로 쉽게 이해하기 위해 설명하자면, Http URI에는 자원의 이름을 명시하고 Http Method를 통해 자원의 CRUD 동작을 결정하는 것이다.

예를 들자면, 학생에 대한 정보를 조회하는 웹 페이지가 있다면 URI에는 이에 해당하는 명사인 students, Method에는 동사인 Read(GET)이 적용되어야 한다는 뜻이다.(설명이 어렵네..)

이러한 REST를 기반으로 API를 구현한 것이 REST API라고 할 수 있겠다.



## 1. django-rest-framework 설치

Django로 REST API를 만드는 것을 쉽게 해주는 django-rest-framework 패키지를 설치한다.

```bash
pip install djangorestframework
```

이를 통해 구현된 api의 document를 만들 수 있는 django-rest-swagger도 설치한다.

```bash
pip install django-rest-swagger
```

이 패키지들을 settings.py에 추가한다.

```python
# settings.py
...

INSTALLED_APPS = [
    'rest_framework',
    'rest_framework_swagger',
    ...
]
```

pip를 통해 설치할 때의 패키지 명과 settings.py에 추가할 때의 app 명이 다르다.



## 2. api 앱 생성

위에서 설치한 패키지들을 활용할 api 앱을 생성하고 settings.py에 추가한다.

```bash
python manage.py startapp api
```

```python
# settings.py
...

INSTALLED_APPS = [
	...
	'api',
]
```



## 3. urls.py 구성

api 요청이 들어올 url을 정의한다.

일단 영화 장르 정보를 받을 url만 구성해보도록 하자.

```python
# watcha/urls.py
...
from django.urls import path, include

urlpatterns = [
    ...
    path('api/', include('api.urls')),
]
```

```python
# api/urls.py
from django.urls import path
from . import views
from rest_framework_swagger.views import get_swagger_view

app_name = 'api'

urlpatterns = [
    path('genres/', views.genre_list, name='genre_list'),
    
    path('docs/', get_swagger_view(title='API')),
]
```



## 4. serializers.py 구성

이제 위에서 설치한 djangorestframework를 사용할 때이다.

여기서 제공하는 serializers를 통해 간편하게 DB의 내용물을 json 형태로 변환하여 응답할 수 있게 만든다.

api 앱 내에 serializers.py를 생성하고 serializer를 구성한다.

```python
# api/serializers.py
from rest_framework import serializers
from movies.models import Genre, Movie, Score

class GenreSerializer(serializers.ModelSerializer):
    class Meta:
        model = Genre
        fields = ['id', 'name', 'genreId']
```

GenreSerializer 클래스를 만들고 메타 클래스에서 해당 serializer에 대응될 모델과 제공될 데이터 컬럼인 fields를 정의해준다.



## 5. views.py 구성

이제 serializer를 통해 json형태의 응답을 하도록 views.py를 구성한다.

```python
# api/views.py
from django.shortcuts import render
from movies.models import Genre, Movie, Score
from .serializers import GenreSerializer
from rest_framework.decorators import api_view
from rest_framework.response import Response

@api_view(['GET'])
def genre_list(request):
    genres = Genre.objects.all()
    serializer = GenreSerializer(genres, many=True)
    return Response(serializer.data)
```

api_view 데코레이터를 통해 요청받을 Method를 정한다. 위에서는 GET방식의 Method만을 받아 응답한다.

genres에 DB에 저장된 모든 장르를 가져온다.

이를 GenreSerializer에 매칭시켜 serializer에 저장한다. 모든 장르를 가져왔으므로 many=True 옵션을 넣어준다.

serializer.data를 반환하여 응답한다.



이제 서버를 실행시키고 `BASE_URL/api/genres`로 접속하여 결과를 확인한다.

![django_api_genres1](/assets/img/django/django_api_genres1.jpg)



평소에 보던 json과는 조금 달라보인다. 이는 Response라는 객체를 이용하여 응답을 보냈기 때문이다. 이를 일반적인 json의 형태로 응답하고 싶다면 views.py를 수정해야 한다.

```python
# api/views.py
...
from django.http import JsonResponse
import json

@api_view(['GET'])
def genre_list(request):
    genres = Genre.objects.all()
    serializer = GenreSerializer(genres, many=True)
    return JsonResponse(json.loads(json.dumps(serializer.data)), safe=False)
```

OrderedDict 객체인 serializer.data를 dictionary로 변환하기 위해 json.loads()와 json.dumps를 이용했다. 그리고 serializer의 옵션에 many=True가 적용되었으므로 이렇게 변환한 내용은 리스트이다. JsonResponse 객체를 통해 데이터를 json 형태로 반환할 때는 딕셔너리 형태의 인자가 주어져야 하는데 위에서는 리스트가 인자로 주어졌기 때문에 safe=False 옵션을 추가하여 반환이 가능하도록 설정한다.

![django_api_genres2](/assets/img/django/django_api_genres2.jpg)