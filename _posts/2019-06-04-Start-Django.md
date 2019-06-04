---
layout: post
title: "[Django] Django 시작하기"
author: Hoik Jang
categories: Django
comments: true
sitemap:
  changefreq: daily
  priority: 1.0
---

이제 새로 만든 AWS EC2 인스턴스 안에서 Django를 사용하여 웹 어플리케이션을 만들어볼 것이다.

일단 Django란 Python으로 만들어진 무료 오픈소스 웹 어플리케이션 프레임워크이다. Django라고 하는 웹 사이트를 만드는데 필요한 요소(로그인, 회원가입, 폼 등등)를 갖춘 틀을 이용하여 쉽고 빠르게 웹 사이트를 만들 수 있다.
Django를 설치하고 사용하기 전에 pyenv 패키지를 먼저 설치하여 Python의 버전 관리 및 설치를 쉽고 빠르게 할 수 있도록 할 것이다. 그리고 virtualenv를 통해 하나의 인스턴스 안에 가상 환경을 만들어 여러개의 Django 프로젝트를 진행할 수 있도록 만들 것이다.

## 1. pyenv 설치

우선 pyenv를 먼저 설치한다.

앞서 배웠던 PuTTY를 통해 EC2 인스턴스에 접속한 뒤 command shell에 차례대로 아래의 내용을 입력한다.

```shell
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bashrc
exec "$SHELL"
git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bashrc
exec "$SHELL"
```

위의 내용을 통해 pyenv가 인스턴스 내에 설치되었다.

이제 pyenv를 통해 Python을 설치한다.

```shell
pyenv install 3.6.7
pyenv global 3.6.7
```

여기서 3.6.7은 Python의 버전을 의미하며 다른 버전을 설치하고 싶다면

```shell
pyenv install -list
```

를 입력하여 설치 가능한 Python 버전을 확인할 수 있다.

Python 버전은 3.6.x 이상을 설치하도록 하자.

global은 Python 명령을 수행할 때 기본적으로 실행되는 Python 버전을 설정해주는 명령이다.



## 2. 프로젝트 생성

이제 Django 프로젝트를 진행할 폴더를 만든다.

나는 인스타그램처럼 사진과 글을 포스팅할 수 있는 웹사이트를 만들어볼 것이기 때문에 폴더의 이름은 INSTAGRAM으로 짓겠다.

```shell
mkdir INSTAGRAM
```

그리고 해당 폴더에 진입하여 이 폴더를 가상 환경으로 만든다.

```shell
cd INSTAGRAM

pyenv virtualenv 3.6.7 instagram-venv
pyenv local instagram-venv
```

virtualenv를 통해 instagram-venv라는 가상 환경을 만들고 local 명령어를 통해 이 가상 환경이 항상 activation 되도록 설정한다.

가상 환경으로 설정해주면 이 폴더 안과 밖은 전혀 다른 공간이 된다고 생각하면 편하다.

(예를 들어 폴더 안을 가상 환경으로 설정한 후 Django를 설치해도 이 폴더 밖에는 Django가 설치되어 있지 않다.)

가상 환경을 설정한 후 Django를 설치한다.

```shell
pip install django==2.1.8
```

2.1.8 버전의 Django를 설치하는 명령어이다. 최근에 2.2 버전대가 release 되면서 바뀐 내용들이 있는데 나는 그 전에 공부한 내용을 바탕으로 포스팅을 할 것이기 때문에 2.1.8 버전을 설치했다.

이제 Django 프로젝트를 시작해보자.

```shell
django-admin startproject instagram .
```

startproject라는 명령어를 통해 Django 프로젝트를 시작할 수 있다.

프로젝트가 생성되면 프로젝트 폴더 내부에 다음과 같은 트리가 형성된다.

```
.
├── db.sqlite3
├── instagram
│   ├── __init__.py
│   ├── __pycache__
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py
```

이제 Django가 잘 설치되어 구동되는지 확인해보자.

```shell
python manage.py runserver 0.0.0.0:8000
```

브라우저에 `http://[퍼블릭 DNS 도메인]:8000`을 입력하여 서버가 잘 실행되는지 확인한다.

아래와 같은 화면이 출력되면 성공적인 Django의 첫 발이 된 것이다.

![django_first_runserver](/assets/img/django/django_first_runserver.jpg)