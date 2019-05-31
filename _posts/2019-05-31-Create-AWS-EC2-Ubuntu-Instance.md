---
layout: post
title: "[AWS] EC2를 이용하여 Ubuntu 인스턴스 만들기"
author: Hoik Jang
categories: AWS
comments: true
---

나는 Python 기반의 웹 프레임워크인 Django를 배우면서 C9을 이용해 공부하고 배포했다.

그런데 C9 서비스가 6월에 종료된다는 소식을 들었고 그럼 어디서 Django 공부를 해야할지 고민을 하다가, 어차피 AWS 공부 해야하니까 AWS에서 인스턴스를 만들고 배포하는 연습을 하자고 결정했다.

그래서 지금부터는 AWS에 회원가입을 하면 1년 동안 무료로 사용할 수 있는 EC2에 대한 간략한 사용법을 써볼까 한다.



## 1. EC2 인스턴스 생성

[AWS 사이트](<https://aws.amazon.com/ko/>)에 들어가서 회원가입을 한 후 콘솔에 로그인을 하자.

사이트 상단 메뉴의 서비스 탭에서 EC2를 찾아 들어간다.

![aws_ec2](/assets/img/aws/aws_ec2.jpg)

그러면 EC2 대시보드 페이지로 이동할 것이고 여기서 인스턴스 시작 버튼을 누른다.

여기서 인스턴스는 aws에서 제공하는 원격 컴퓨터라고 생각하자.

![aws_ec2_instance_start](/assets/img/aws/aws_ec2_instance_start.jpg)

AMI(Amazon Machine Image)를 선택한다.

프리 티어 사용 가능이라 적힌 AMI가 무료로 이용할 수 있는 종류이다.

나는 Ubuntu Server 16.04 LTS를 선택했다.

![aws_ec2_ami](/assets/img/aws/aws_ec2_ami.jpg)

인스턴스 유형을 선택한다.

프리 티어 사용 가능이라 적힌 유형을 선택하고 오른쪽 아래 다음 버튼을 클릭한다.

![aws_ec2_instance_type](/assets/img/aws/aws_ec2_instance_type.jpg)

인스턴스 세부 정보 구성은 그대로 두고 다음 버튼을 클릭한다.

스토리지 추가에서 크기 탭은 최대 30GB까지 지정할 수 있고 리눅스 기반의 인스턴스는 디폴트로 8GB가 할당된다. 그대로 두고 다음 버튼을 클릭한다.

태그 추가 역시 그대로 두고 다음 버튼을 클릭한다.

보안 그룹 구성에는 기본적으로 ssh 접속을 위해 22번 포트가 설정되어 있다. 외부 사용자가 이 서버에 접속하여 서비스를 이용할 수 있도록 하기 위해 http 유형의 80번 포트와 8000번 포트를 열어둔다.

규칙 추가 버튼으로 눌러 아래 사진과 같이 그룹을 추가한다.

![aws_ec2_security_group](/assets/img/aws/aws_ec2_security_group.jpg)

검토 및 시작 버튼을 클릭한다.

이후 시작하기 버튼을 클릭하면 다음과 같은 팝업이 활성화 될 것이다.

인스턴스를 처음으로 생성하는 것이므로 새 키 페어 생성을 선택하고 키 페어 이름(.pem 파일의 이름)을 입력하고 키 페어 다운로드 버튼을 클릭한다.

![aws_ec2_key](/assets/img/aws/aws_ec2_key.jpg)

그러면 .pem 파일이 다운받아질 것이고 이를 안전한 위치로 옮긴 후 인스턴스 시작 버튼을 누르면 새로운 EC2 인스턴스 생성이 완료된다.



다음에는 Windows 운영체제 내에서 위의 인스턴스에 접속하는 방법을 포스팅하겠다.