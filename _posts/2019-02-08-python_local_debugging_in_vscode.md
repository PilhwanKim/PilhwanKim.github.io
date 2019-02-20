---
layout: default
title:  "Vs Code 에서 Python Remote 디버깅 세팅"
categories: main
tags: [python, debugging, VSCode]
---

# Vs Code Python Remote 디버깅 세팅

## 목적

- 이미 마련된 개발용 또는 QA 서버 환경을 그대로 활용하여 디버깅 할 수 있도록 함

## 필요한 확장 플러그인 설치

- Python
  - 파이썬 인터프리터 실행을 위한 플러그인
- Project Manager
  - VSCODE 의 각 폴더를 프로젝트로 등록하면 리스트로 관리 및 자동 스위치 가능함
- sftp
  - 서버에 버전 관리 시스템(Version control system) 없이도 코드를 Upload 하여 실행할 수 있도록 함

## 확장 플러그인 별 설정

- Project Manager
  - File > Open Folder > [프로젝트 디렉토리] 선택
  - shift + ctrl + p > Project Manager: save project
  - 프로젝트 명 입력하여 등록
  - 왼쪽 사이드바 > 프로젝트 메니저 > 포르젝트 목록 선택
  - 각 프로젝트로 변경됨
- sftp
  - shift + ctrl + p > SFTP:config
  - 다음과 같이 추가
    ```json
        [
            {
                "protocol": "sftp",
                "name": "service-test",
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
    - protocol : 'SFTP' 로 고정
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
  - 기본 디버그 세팅이 생성됨
  - .vscode/launch.js 파일 설정을 아래와 같이 변경함
    ```json
    {
        // IntelliSense를 사용하여 가능한 특성에 대해 알아보세요.
        // 기존 특성에 대한 설명을 보려면 가리킵니다.
        // 자세한 내용을 보려면 https://go.microsoft.com/fwlink/?linkid=830387을(를) 방문하세요.
        "version": "0.2.0",
        "configurations": [
            {
                "name": "service test django(2.7)",
                "type": "python",
                "request": "attach",
                "host": "1.1.1.1",
                "port": 5678,
                "localRoot": "${workspaceFolder}",
                "remoteRoot": "/tmp/api",
                "pythonPath": "/usr/local/py2.7/bin/python",
                "console": "integratedTerminal",
                "program": "${workspaceFolder}/manage.py",
                "args": [
                    "runserver",
                    "--noreload",
                    "--nothreading"
                ],
                "django": true
            },
            {
                "name": "service test celery(2.7)",
                "type": "python",
                "request": "attach",
                "host": "1.1.1.1",
                "port": 5678,
                "localRoot": "${workspaceFolder}",
                "remoteRoot": "/tmp/api",
                "pythonPath": "/usr/local/py2.7/bin/python",
                "console": "integratedTerminal",
                "program": "${workspaceFolder}/manage.py",
                "args": [
                    "celery",
                    "worker",
                    "-E",
                    "-l",
                    "INFO",
                    "--pool=solo",
                ],
                "django": true
            },
            {
                "name": "service test unittest(2.7)",
                "type": "python",
                "request": "attach",
                "host": "1.1.1.1",
                "port": 5678,
                "localRoot": "${workspaceFolder}",
                "remoteRoot": "/tmp/api",
                "pythonPath": "/usr/local/py2.7/bin/python",
                "console": "integratedTerminal",
                "program": "${workspaceFolder}/manage.py",
                "args": [
                    "test",
                    "app.tests.testUser.TestUserData.test_msg",
                ],
                "django": true
            },
            {
                "name": "service test pytest(2.7)",
                "type": "python",
                "request": "attach",
                "host": "1.1.1.1",
                "port": 5678,
                "localRoot": "${workspaceFolder}",
                "remoteRoot": "/tmp/api",
                "pythonPath": "/usr/local/py2.7/bin/python",
                "console": "integratedTerminal",
                "program": "pytest",
                "args": [
                    "tests/unittest/test_mail.py::test_certi_mail_send"
                ]
            },
            {
                "name": "service test python(2.7)",
                "type": "python",
                "request": "attach",
                "host": "1.1.1.1",
                "port": 5678,
                "localRoot": "${workspaceFolder}",
                "pythonPath": "/usr/local/py2.7/bin/python",
                "remoteRoot": "/tmp/api",
                "program": "chat/mailSendQueue.py",
                "console": "integratedTerminal",
            }
        ]
    }
    ```

  - 옵션 설명
    - name : 디버그 선택 메뉴 이름
    - type : 디버깅 종류. 'python' 고정.
    - request : 디버깅 방식. remote debugging 은 'attach'
    - localRoot : 개발 PC 에서 소스 위치. ${workspaceFolder}" 를 루트로 두므로 변경 불필요
    - remoteRoot : 리모트 디버깅 프로세스가 실행되는 서버 주소
    - host : 리모트 디버깅 접속할 호스트 명
    - port : 리모트 디버깅 접속할 포트
    - pythonPath : python interpreter 의 Path
    - program : python 실행할 코드
    - args : python 실행시 argument
    - console : 실행할 콘솔 방식 선택. "integratedTerminal" 로 고정
    - django : django 프레임워크 실행시 true 지정.

## pip install

```sh
# python 2.7 일 경우
pip install more-itertools==5.0.0 pytest-django pytest-profiling

# python 3 이상
pip install pytest-django
```

### python 원격(서버) 실행 명령

  - django 원격 디버깅 실행
  /usr/local/py2.7/bin/python -m ptvsd --host 0.0.0.0 --port 5678 manage.py runserver 0.0.0.0:8000 --noreload --nothreading

  - celery 원격 디버깅 실행
  /usr/local/py2.7/bin/python -m ptvsd --host 0.0.0.0 --port 5678 manage.py celery worker -E -l INFO --pool=solo

  - unitservice test 원격 디버깅 실행
  /usr/local/py2.7/bin/python -m ptvsd --host 0.0.0.0 --port 5678 --wait manage.py service test app.tests.testUser.TestUserData.test_msg

  - pyservice test 원격 디버깅 실행
  /usr/local/py2.7/bin/python -m ptvsd --host 0.0.0.0 --port 5678 --wait -m pyservice test app/tests/test_user.py::test_is_fanclub_staff

  - pure python 원격 디버깅 실행
  /usr/local/py2.7/bin/python -m ptvsd --host 0.0.0.0 --port 5678 chat/mailSendQueue.py

### 디버깅 절차

1. 위에 언급한 대로 VSCODE 의 프로젝트 설정을 마친다.
2. 해당 리모트 디버깅 환경(서버) 으로 ssh 접속
3. 개발용 컴퓨터의 VSCODE 에서 프로젝트 전체를 서버에 업로드(or sync)
4. 해당 서버에 SSH 로그인
5. remoteRoot 로 접근하여 [python 원격(서버) 실행 명령] 으로 디버그용 python 프로세스 띄움
6. 개발용 컴퓨터의 VSCODE 에서 breakpoint 지정
7. 개발용 컴퓨터의 VSCODE 에서 왼쪽 사이드 메뉴 > 디버그 > 선택 > Debug 실행
8. 제대로 접속되었다면 브레이크 포인트로 멈춤
