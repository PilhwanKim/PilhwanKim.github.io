---
layout: default
title:  "Vs Code 에서 Python Remote 디버깅 세팅"
categories: main
tags: [python, debugging, VSCode]
---

# Vs Code Python Remote 디버깅 세팅

## 목적

- 이미 마련된 Test 또는 QA 서버 환경을 그대로 활용하여 디버깅 할 수 있도록 함

## 리모트(서버) 환경 pip install

```sh
# python 2.7 일 경우
pip install ptvsd more-itertools==5.0.0 pytest-django pytest-profiling

# python 3 이상
pip install ptvsd pytest-django pytest-profiling
```

## 개발 PC에 VS Code 설치

- https://code.visualstudio.com/ 이동
- Downloads For Windonws 클릭
- 다운로드된 파일 실행
 
## 필요한 확장 플러그인 설치

- Ctrl + Shift + x > plugin search input 에 아래 각각의 플러그인 검색 > 선택 > install 버튼
- 플러그인 설치 목록
  - Python
    - 파이썬 인터프리터 실행을 위한 플러그인
  - sftp
    - 리모트(서버)에 코드를 Upload 하기 위함
  - Project Manager
    - VSCODE 의 각 폴더를 프로젝트로 등록하면 관리 및 선택하여 자동으로 스위치 가능

## Project Manager 플러그인 : 프로젝트 별로 선택 가능하도록 설정

- File > Open Folder > [프로젝트 디렉토리] 선택
- Ctrl + Shift + p > Project Manager: save project
- 프로젝트 명 입력하여 등록
- 왼쪽 사이드바 > 프로젝트 메니저 > 포르젝트 목록 선택
- 각 프로젝트로 변경됨

## SFTP 플러그인 : 리모트 환경에 파이썬 코드 쉽게 올리기

- Ctrl + Shift + p > SFTP:config
- 다음과 같이 추가

```json
[
    {
        "protocol": "sftp",
        "name": "service-qa",
        "host": "{inpput your remote address}",
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

## 디버그 대상 Python 코드 추가

- 일반 python 실행시

```python
import ptvsd
ptvsd.enable_attach(redirect_output=True)
```

- Django 실행(manage.py 에 추가)

```python
import os
import sys

if __name__ == "__main__":
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myproject.settings")

    from django.core.management import execute_from_command_line

    import ptvsd
    ptvsd.enable_attach(redirect_output=True)

    execute_from_command_line(sys.argv)
```

- pytest-django 적용시(conftest.py 에 추가)

```python
import pytest

def pytest_configure(config):
    import ptvsd
    ptvsd.enable_attach(redirect_output=True)

```

## 리모트 디버그 메뉴 설정

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
            // django 디버그 설정
            {
                "name": "Lysn QA django(2.7)",
                "type": "python",
                "request": "attach",
                "host": "{inpput your remote address}",
                "port": 5678,
                "localRoot": "${workspaceFolder}",
                "remoteRoot": "/tmp/api",
                "pythonPath": "{your remote python path}",
                "console": "integratedTerminal",
                "program": "${workspaceFolder}/manage.py",
                "args": [
                    "runserver",
                    "--noreload",
                    "--nothreading"
                ],
                "django": true
            },
            // django celery 디버그 설정
            {
                "name": "Lysn QA celery(2.7)",
                "type": "python",
                "request": "attach",
                "host": "{inpput your remote address}",
                "port": 5678,
                "localRoot": "${workspaceFolder}",
                "remoteRoot": "/tmp/api",
                "pythonPath": "{your remote python path}",
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
            // django unittest 디버그 설정
            {
                "name": "Lysn QA unittest(2.7)",
                "type": "python",
                "request": "attach",
                "host": "{inpput your remote address}",
                "port": 5678,
                "localRoot": "${workspaceFolder}",
                "remoteRoot": "/tmp/api",
                "pythonPath": "{your remote python path}",
                "console": "integratedTerminal",
                "program": "${workspaceFolder}/manage.py",
                "args": [
                    "test",
                    "app.tests.testUser.TestUserData.test_msg",
                ],
                "django": true
            },
            // pytest-django  디버그 설정
            {
                "name": "Lysn QA pytest(2.7)",
                "type": "python",
                "request": "attach",
                "host": "{inpput your remote address}",
                "port": 5678,
                "localRoot": "${workspaceFolder}",
                "remoteRoot": "/tmp/api",
                "pythonPath": "{your remote python path}",
                "console": "integratedTerminal",
                "program": "pytest",
                "args": [
                    "tests/unittest/test_mail.py::test_certi_mail_send"
                ]
            },
            // 일반 python 디버그 설정
            {
                "name": "Lysn QA python(2.7)",
                "type": "python",
                "request": "attach",
                "host": "{inpput your remote address}",
                "port": 5678,
                "localRoot": "${workspaceFolder}",
                "pythonPath": "{your remote python path}",
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
  - django : django 프레임워크 실행에만 true 지정함

## python 원격(서버) 실행 명령

  ```bash
  # python 원격 디버깅 실행
  python -m ptvsd --host 0.0.0.0 --port 5678 app/main.py

  # django 원격 디버깅 실행
  python -m ptvsd --host 0.0.0.0 --port 5678 manage.py runserver 0.0.0.0:8000 --noreload --nothreading
  
  # django unittest 원격 디버깅 실행
  python -m ptvsd --host 0.0.0.0 --port 5678 --wait manage.py test app.tests.testUser.TestUserData.test_msg
  
  # pytest-django 원격 디버깅 실행
  python -m ptvsd --host 0.0.0.0 --port 5678 --wait -m pytest app/tests/test_user.py::test_msg

  # celery 원격 디버깅 실행
  python -m ptvsd --host 0.0.0.0 --port 5678 manage.py celery worker -E -l INFO --pool=solo
  ```

## 리모트 디버깅 절차

1. 위에 설명한대로 개발용 컴퓨터의 VSCODE 프로젝트 설정
2. VSCODE 에서 프로젝트 전체를 sftp 플러그인 기능으로 리모트(서버)의 remoteRoot에 업로드
   1. Ctrl + Shift + E > 프로젝트 파일 전체 선택 > 오른쪽 클릭(메뉴) > upload 선택
3. 리모트(서버)로 SSH 로그인
4. 위에 설정된 remoteRoot 로 가서 [python 원격(서버) 실행 명령] 디버그용 python 프로세스 띄움
5. 정상적으로 실행되면 VSCODE 로 돌아와서 breakpoint 지정
   1. 원하는 python 코드에서 editor 화면 line number 가 있는 왼쪽에 클릭하면 지정됨
6. VSCODE 에서 디버그 실행
   1. VSCODE 왼쪽 사이드 메뉴 > [디버그] 선택 > [디버그 종류] 선택 > Debug 실행
7. 브레이크 포인트에서 멈추면 성공!

## pytest-profile 사용법

- Test 결과에서 각 함수별 시간 측정 결과를 추가로 보여줌

```sh
Profiling (from /tmp/api/prof/combined.prof):
Thu Feb 21 12:26:26 2019    /tmp/api/prof/combined.prof

         22532 function calls (22337 primitive calls) in 0.095 seconds

   Ordered by: cumulative time
   List reduced from 819 to 20 due to restriction <20>

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.095    0.095 runner.py:118(pytest_runtest_call)
        1    0.000    0.000    0.095    0.095 python.py:1436(runtest)
        1    0.000    0.000    0.095    0.095 hooks.py:270(__call__)
        1    0.000    0.000    0.095    0.095 manager.py:65(_hookexec)
        1    0.000    0.000    0.095    0.095 manager.py:59(<lambda>)
        1    0.000    0.000    0.095    0.095 callers.py:157(_multicall)
        1    0.000    0.000    0.095    0.095 python.py:156(pytest_pyfunc_call)
        1    0.000    0.000    0.095    0.095 test_api.py:60(test_signInWithCertiKeyNew)
        1    0.000    0.000    0.092    0.092 csrf.py:57(wrapped_view)
        1    0.000    0.000    0.092    0.092 decorators.py:37(wrapper_function)
        1    0.000    0.000    0.091    0.091 views.py:50(signInWithCertiKeyNew)
        1    0.000    0.000    0.091    0.091 accountApi.py:183(sign_in)
        1    0.000    0.000    0.069    0.069 accountApi.py:2056(onSignIn)
       65    0.000    0.000    0.032    0.000 utils.py:97(inner)
        1    0.000    0.000    0.024    0.024 UserDB.py:397(loginSucceeded)
        4    0.000    0.000    0.022    0.006 query.py:552(update)
       55    0.002    0.000    0.020    0.000 client.py:257(execute_command)
        1    0.000    0.000    0.020    0.020 UserRDB.py:378(_loginSucceeded)
        5    0.000    0.000    0.017    0.003 transaction.py:196(__exit__)
        5    0.000    0.000    0.016    0.003 base.py:167(commit)
```

- argument 추가(--profile) 하여 실행함

```sh
python -m ptvsd --host 0.0.0.0 --port 5678 --wait -m pytest app/tests/test_user.py::test_is_fanclub_staff --remote-debug --profile
```
