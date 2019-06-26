---
layout: post
title: "[Django] Project02 - 새로운 프로젝트 생성 및 환경 구성"
author: Hoik Jang
categories: Django
comments: true
sitemap:
  changefreq: daily
  priority: 1.0
---

첫 번째 개인 프로젝트인 인스타그램 따라하기에 이어 이번에는 영화 추천 웹 페이지를 만들어보고자 한다.

인스타그램 웹 페이지를 만들 때에는 한 가지 기능을 구현할 때마다 새로운 요청 url을 만들었다. 그리고 그 기능의 수행이 완료되면 리디렉션을 하는 구간이 굉장히 많았다.

Django만을 이용했기 때문에 발생하는 이런 불편하고 깔끔하지 못한 상황을 어느 정도 해결하기 위해 이번에는 Vue.js를 이용하여 하나의 url로 CRUD(Create, Read, Update, Delete) 동작이 가능하도록 만들어볼 것이다.



## 1. 프로젝트 생성

새로운 프로젝트를 생성하기 위해 가상 환경을 구성한다.

```shell
mkdir WATCHA
cd WATCHA

pyenv virtualenv 3.6.7 watcha-venv
pyenv local watcha-venv

pip install django==2.1.8

django-admin startproject watcha .
python manage.py startapp accounts
python manage.py startapp movies
```

가상 환경을 구성하고, Django를 설치한 후 프로젝트를 시작한 뒤 accounts 앱과 movies 앱을 생성했다.



## 2. settings.py 환경 설정

```python
# watcha/settings.py
...

ALLOWED_HOSTS = ['*']

INSTALLED_APPS = [
    ...
    'accounts',
    'movies',
]

...

LANGUAGE_CODE = 'ko-kr'

TIME_ZONE = 'Asia/Seoul'
```

INSTALLED_APPS에 위에서 생성한 앱을 추가한다.



## 3. models.py 모델 구성

영화 관련 웹 페이지를 만들어볼 것이기 때문에 movies 앱 내의 models.py에 장르, 영화, 평점에 대한 모델을 구성한다.

```python
# models.py
from django.db import models
from django.contrib.auth import get_user_model

class Genre(models.Model):
    name = models.CharField(max_length=30)
    genreId = models.IntegerField()

    def __str__(self):
        return f"{self.id}: {self.genreId} - {self.name}"

class Movie(models.Model):
    movieNm = models.CharField(max_length=100)
    openDt = models.CharField(max_length=20)
    audiAcc = models.IntegerField()
    image_url = models.CharField(max_length=200)
    description = models.TextField()
    genres = models.ManyToManyField(Genre, related_name='movie', blank=True)
    showTm = models.IntegerField()

    def __str__(self):
        return f"{self.movieNm}"

class Score(models.Model):
    content = models.CharField(max_length=100)
    score = models.IntegerField()
    movie = models.ForeignKey(Movie, related_name="scores", on_delete=models.CASCADE)
    user = models.ForeignKey(get_user_model(), related_name="scores", on_delete=models.CASCADE)

    def __str__(self):
        return f"{self.movie.movieNm}: {self.content} - {self.score}"
```

이 모델을 데이터베이스에 마이그레이션 한다.

```shell
python manage.py makemigrations
python manage.py migrate
```



## 4. seed data 입력

영화에 대한 정보를 보여주는 페이지를 만들 것이기 때문에 그에 대한 seed data가 필요하다.

나는 이 seed data를 만들기 위해 python 코드를 짜서 seed data를 json 파일로 만들었다.

실제 데이터를 반영하기 위해 [영화진흥위원회](<http://www.kobis.or.kr/kobisopenapi/homepg/main/main.do>)와 [TMDb](<https://developers.themoviedb.org/>)에서 제공하는 Open API를 적절히 조합하여 구성했다.
이렇게 만든 json 파일의 구성요소는 아래와 같다.

```json
// genres.json
[
	{
		"pk": 1,
		"model": "movies.genre",
		"fields": {
			"genreId": 28,
			"name": "액션"
		}
	},
    ...
]
```

```json
// movies.json
[
	{
		"pk": 1,
		"model": "movies.movie",
		"fields": {
			"movieNm": "토이 스토리 4",
			"openDt": "2019-06-20",
			"audiAcc": 1115626,
			"image_url": "https://image.tmdb.org/t/p/w500/b0XF0LzpKLaSVsY9dlUIZP2BdT6.jpg",
			"description": "장난감의 운명을 거부하고 떠난 새 친구 ‘포키’를 찾기 위해 길 위에 나선 ‘우디’는 우연히 오랜 친구 ‘보핍’을 만나고 그녀를 통해 새로운 세상에 눈을 뜨게 된다. 한편, ‘버즈’와 친구들은 사라진 ‘우디’와 ‘포키’를 찾아 세상 밖 위험천만한 모험을 떠나게 되는데… 당신이 기다려온 그들의 진짜 이야기가 공개된다!",
			"genres": [
				2,
				3,
				4,
				8
			],
			"showTm": 99
		}
	},
    ...
]
```

pk는 해당 데이터가 가질 primary key이고, model은 이 데이터가 적용될 모델이다. 이 영화 데이터는 movies 앱에 있는 Genre와 Movie 모델에 적용될 것이기 때문에 movies.genre, movies.movie로 한다.

fields는 해당 영화의 데이터 부분으로 Genre와 Movie 모델의 필드명과 같은 데이터들이 입력된다.

이렇게 데이터를 구성한 뒤 이 데이터를 데이터베이스에 저장한다.

```shell
python manage.py loaddata genres.json
python manage.py loaddata movies.json
```



다음 포스트에는 rest_framework와 rest_framework_swagger라는 패키지를 이용하여 위의 데이터들을 볼 수 있는 api 서버를 구성해볼 것이다.