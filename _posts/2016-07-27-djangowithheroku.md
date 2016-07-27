---
published: true
title: Heroku 에 Django Project 올리기
layout: post
---
### Heroky Starter Template 으로  Django 개발

1. template 다운 받기  
`https://github.com/heroku/heroku-django-template`

2. 아예 처음부터 template 기반으로 시작하기

```
$ django-admin.py startproject --template=https://github.com/heroku/heroku-django-template/archive/master.zip --name=Procfile helloworld

$ pip freeze > requirement.txt  # 설치 패키지 정보를 파일로 남김
```
* runtime.txt 를 수정하여 python 버전 수정할 것

### Heroku Toolbelt 설치
`https://toolbelt.heroku.com/`

#### 설치 후 heroku CLI 로그인
```
$ heroku login
Enter your Heroku credentials.
Email: adam@example.com
Password (typing will be hidden):
Authentication successful.
```

#### heroku app 만들기
```
$ cd ~/myapp
$ heroku create chiyodad # 이름을 지정해주지 않으면 안드로메다스러운 이름으로 지정됨
Creating stark-fog-398... done, stack is cedar-14
http://stark-fog-398.herokuapp.com/ | https://git.heroku.com/stark-fog-398.git
Git remote heroku added

$ git remote -v  # heroku 가 remote 저장소로 설정되어 있는지 확인

heroku	https://git.heroku.com/chiyodad.git (fetch)
heroku	https://git.heroku.com/chiyodad.git (push)

```

* cli 설치 경로
`/usr/local/heroku and /usr/local/heroku/bin will be added to your PATH.`


### Heroku 에서 python 명령어 수행하기

```
$ python manage.py makemigrations # heroky 에 올리기 전 migrations 파일 생성
$ git add .
$ git commit -m 'initial commit'
$ git push heroku master
```

Deploy 와 Django 관련 패키지가 자동을 설치됨
DB Object는 생성이 안됐기 때문에 온라인으로 migrate 를 해준다.

```
$ heroku run python manage.py migrate
$ heroku run python manage.py createsuperuser  # 사용자 생성
$ heroku open  # 접속 확인
```
