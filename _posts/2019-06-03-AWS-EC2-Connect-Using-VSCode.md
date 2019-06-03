---
layout: post
title: "[AWS] Visual Studio Code를 이용하여 EC2 인스턴스 내 파일 수정하기"
author: Hoik Jang
categories: AWS
comments: true
---

이제 EC2 인스턴스에 접속은 했는데 켜진 것이라고는 터미널창 하나밖에 없다.

이 터미널에서 프로젝트를 생성하고 코드를 작성하기에는 좀 무리가 있다.

그래서 Visual Studio Code를 EC2 인스턴스에 연결하여 코드를 작성하고 수정할 것이다.



## 1. ftp-simple 설치

[Visual Studio Code](<https://code.visualstudio.com/download>)를 실행한 후, 왼쪽의 Extensions 탭에서 ftp를 검색하여 ftp-simple 플러그인을 설치한다.

![aws_vscode_ftp_install](/assets/img/aws/aws_vscode_ftp_install.jpg)

`F1`키를 눌러 Command Pallette를 열고 ftp-simple을 입력한 후 ftp-simple: Config - FTP connection setting을 클릭하면 세팅 파일이 실행된다.

```json
// ftp-simple-temp.json
[
	{
		"name": "localhost",
		"host": "",
		"port": 21,
		"type": "ftp",
		"username": "",
		"password": "",
		"path": "/",
		"autosave": true,
		"confirm": true,
	}
]
```

이 파일을 수정하여 EC2 인스턴스에 접속할 수 있도록 만든다.

```json
// ftp-simple-temp.json
[
	{
		"name": "AWS EC2 Ubuntu", // 이 ftp 연결의 이름을 지어준다.
		"host": "111.222.333.444", // 인스턴스에 접속하는 ip를 입력한다.
		"port": 22, // ssh port를 22번으로 세팅한 경우 22를 입력한다.
		"type": "sftp",
		"username": "ubuntu", // PuTTY 설정 때 Host Name 앞에 썼던 username을 입력한다. 보통은 ubuntu나 root
		"password": "",
		"path": "/",
		"autosave": true,
		"confirm": true,
		"privateKey": "C:\\Users\\test.pem" // 보안 키 페어 파일 경로를 입력한다.
	}
]
```

주의할 점은 privateKeyPath가 아니라 privateKey라는 점이다.

파일을 수정하고 저장한 후 VSCode를 재실행한다.

`F1`키를 눌러 ftp-simple를 입력한 후 ftp-simple: Remote directory open to workspace를 클릭한 후 위에서 지정했던 ftp 연결의 name을 클릭하면 상세 directory를 지정하여 열 수 있다. (보통 username이 ubuntu일 경우 PuTTY 실행 시 기본 디렉토리로 지정된 곳은 home/ubuntu 이다.)

![aws_vscode_ftp_config](/assets/img/aws/aws_vscode_ftp_config.jpg)

이제 VSCode를 통해 EC2 인스턴스 내의 코드를 작성하고 수정할 수 있을 것이다.