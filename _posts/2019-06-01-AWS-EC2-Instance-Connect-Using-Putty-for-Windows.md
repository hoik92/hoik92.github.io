---
layout: post
title: "[AWS] PuTTY를 이용하여 EC2 인스턴스에 ssh 접속하기 for Windows"
author: Hoik Jang
categories: AWS
comments: true
---

앞선 포스팅에서 AWS의 EC2 인스턴스를 생성했다.

이제 새로 만든 EC2 인스턴스에 접속을 해야 그 인스턴스 내에서 작업을 할 수 있을 것이다.

SSH 접속을 통해 인스턴스에 접속을 해야하는데 Windows의 cmd 콘솔에서는 SSH 접속이 불가능하다.(Windows10에서는 OpenSSH라는 앱을 추가하여 SSH 접속이 가능하다.)

그래서 Windows 운영체제에서 SSH 접속을 가능하게 해주는 PuTTY라는 프로그램을 설치하여 새로 만든 EC2 인스턴스에 접속할 것이다.

나의 설명이 부족할 수 있으니 AWS에서 제공하는 [인스턴스 연결 방법](<https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/putty.html>)을 참고해도 좋다.



## 1. PuTTY 설치

[PuTTY 다운로드 사이트](<https://www.chiark.greenend.org.uk/~sgtatham/putty/>)에서 PuTTY를 다운받아 설치한다.

구글에 PuTTY를 검색해도 상관없다.



## 2. PuTTYgen을 이용한 프라이빗 키 변환

전 포스팅에서 인스턴스를 생성할 때 막바지에 프라이빗 키를 생성했다.(.pem 파일)

PuTTY는 EC2 인스턴스에 접속할 때 필요한 이 프라이빗 키 파일의 형식(.pem)을 지원하지 않는다. 따라서 PuTTY를 설치하면 생기는 PuTTYgen이라는 도구를 이용해 PuTTY에서 사용가능한 키 형식(.ppk)으로 변환해야 한다.

시작 메뉴에서 PuTTYgen을 검색하여 실행한다.

![aws_putty_puttygen](/assets/img/aws/aws_putty_puttygen.jpg)

PuTTYgen 프로그램 아래 Parameters 탭의 Type of key to generate 메뉴에서 RSA를 선택한다.

그 위의 Actions 탭에서 Load 버튼을 눌러 프라이빗 키 파일(.pem)을 로드한다.

![aws_putty_puttygen2](/assets/img/aws/aws_putty_puttygen2.jpg)

PuTTYgen에서 키 파일을 로드할 때 기본적으로 .ppk 파일만 표시되기 때문에 모든 유형의 파일을 표시할 수 있도록 옵션을 바꾸고 .pem 파일을 로드한다.

![aws_putty_puttygen_load](/assets/img/aws/aws_putty_puttygen_load.jpg)

Load 버튼 아래에 있는 Save private key 버튼을 누른다.

경고 팝업창이 뜨면 예를 선택하고 기존 프라이빗 키 파일(.pem)과 같은 이름으로 .ppk 파일을 저장한다.



## 3. PuTTY를 이용한 EC2 인스턴스 연결

이제 PuTTY를 이용하여 인스턴스에 접속하자.

PuTTY를 실행한다.

Category 창의 Session 탭을 선택한다.
![aws_putty_session](/assets/img/aws/aws_putty_session.jpg)

Host Name에 `user_name@public_dns_name`형식으로 입력한다.

`user_name`은 인스턴스를 생성할 때 선택한 AMI에 적합한 이름을 입력해야 한다.

![aws_putty_session_username](/assets/img/aws/aws_putty_session_username.jpg)

나는 AMI를 Ubuntu로 선택했기 때문에 `user_name`은 `ubuntu`라고 적었다.
`public_dns_name`은 AWS EC2 사이트에서 인스턴스 탭을 클릭해 들어가면 보이는 인스턴스의 퍼블릭 DNS(IPv4)를 그대로 복사하여 입력하면 된다.

![aws_putty_session_dnsname](/assets/img/aws/aws_putty_session_dnsname.jpg)

Host Name 옆에 Port가 22로 적혀있는지 확인한다.

Category 창의 Connection-SSH 탭을 확장한 후 Auth 탭을 선택한다.

Browse 버튼을 눌러 위에서 생성한 프라이빗 키 파일(.ppk)을 로드한다.

![aws_putty_auth](/assets/img/aws/aws_putty_auth.jpg)

다시 Session 탭으로 돌아가서 Saved Sessions 입력창에 지금까지 했던 설정을 저장할 이름을 쓰고 Save 버튼을 누른다.

다음에 같은 인스턴스에 접속할 때 이 설정을 로드하면 번거롭게 다시 입력할 필요없이 바로 연결할 수 있다.

Open 버튼을 눌러 인스턴스에 접속한다.

![aws_putty_shell](/assets/img/aws/aws_putty_shell.jpg)

SSH 접속으로 EC2 인스턴스에 연결된 것을 PuTTY shell을 통해 볼 수 있다.

이제 이 새로운 인스턴스(컴퓨터)에서 하고 싶은 작업을 수행하면 된다.