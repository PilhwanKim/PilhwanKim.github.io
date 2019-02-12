---
layout: default
title:  "Vs Code Python Remote 디버깅 세팅"
categories: main
tags: [python, debugging, VSCode]
---

# Vs Code Python Remote 디버깅 세팅

## 목적

- 이미 마련된 개발용 또는 QA 서버 환경을 그대로 활용하여 디버깅 할 수 있도록 함

## 필요한 확장 플러그인 설치

- Python
  - 파이썬 인터프리터 실행을 위한 플러그인
- sftp
  - 서버에 버전 관리 시스템(Version control system) 없이도 코드를 Upload 하여 실행할 수 있도록 함

## 확장 플러그인 별 설정

- sftp
  - shift + ctrl + p > SFTP:config
  - 다음과 같이 추가
      ```json
          [
              {
                  "protocol": "sftp",
                  "name": "service-qa",
                  "host": "1.1.1.1",
                  "port": 23,
                  "context": "./",
                  "username": "",
                  "password": "",
                  "remotePath": "/tmp/api"
              }
          ]
      ```
  - 옵션 설명
    - name : 호스트 이름 입력
    - host : 호스트 주소 입력
    - port : ssh 포트 입력
    - context : 로컬 환경에서의 파이썬 소스 Root Path
    - username : 원격으로 붙는 곳의 소스 위치. 직접 지정하도록 함
    - password : django 실행시
    - remotePath : ssh로 접근하는 서버의 파이썬 Root Path

## Workspace 설정

- 상단 메뉴 > 파일(F) > 폴더 열기
  - 프로젝트 폴더를 선택
    - 예시) Community\04-Construction\Qa\api
- 상단 메뉴 > 파일(F) > 작업 영역을 다른 이름으로 저장
  - workspace 설정 파일을 원하는 위치에 저장
    - 예시) dev-api.code-workspace

## 디버그 메뉴 설정

- 왼쪽 사이드 메뉴 > 디버그 > 구성 없음 > 구성 추가
  - 'python' 선택
  - 여러 디버그 세팅이 생성됨
- django 프레임워크가 remote interpreter 실행 가능한 세팅 맞추기
  - Django 설정
    ```json
    {
        "name": "Dev API Uwsgi(2.7)",
        "type": "python",
        "request": "attach",
        "localRoot": "${workspaceFolder}",
        "remoteRoot": "/tmp/api",
        "port": 3000,
        "host": "localhost",
        "program": "${workspaceFolder}/manage.py",
        "console": "integratedTerminal",
        "args": [
            "runserver",
            "--noreload",
            "--nothreading"
        ],
        "django": true
    }
    ```
  - pytest 설정
    ```json
    {
        "name": "Dev API Uwsgi(2.7)",
        "type": "python",
        "request": "attach",
        "localRoot": "${workspaceFolder}",
        "remoteRoot": "/tmp/api",
        "port": 3000,
        "host": "localhost",
        "program": "${workspaceFolder}/manage.py",
        "console": "integratedTerminal",
        "args": [
            "runserver",
            "--noreload",
            "--nothreading"
        ],
        "django": true
    }
    ```

  - 옵션 설명
    - name : 디버그 선택 메뉴 이름
    - request : 디버깅 방식. 'attach' 일 경우. remote debugging 가능
    - localRoot : 개발 PC 에서 소스 위치. ${workspaceFolder}" 를 루트로 두므로 변경 불필요.
    - remoteRoot : 리모트 디버깅 프로세스가 실행되는 서버 주소
    - port : 리모트 디버깅 접속을 위한 포트
    - args : python 실행시 argument 들

## pip install

```sh
# python 2.7 일 경우
pip install more-itertools==5.0.0 pytest-django model_mommy

# python 3 부터
pip install pytest-django model_mommy
```