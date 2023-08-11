---
title: "PostgreSQL 초기 설정"
layout: post
categories: summary
tags:
- postgresql
---

## 설치
- `sudo apt update`
- `sudo apt install postgresql postgresql-contrib`
- `sudo systemctl start postgresql.service`

Postgres에서는 `role`이라는 개념으로 authentication과 authorization을 관리한다. `role`의 이름과 동일한 이름을 가진 linux sysmtem account가 있다면 해당 account를 통해 같은 이름을 가진 `role`로 Postgres에 로그인할 수 있다.

설치가 완료되면 `postgres`라는 linux system account가 생성된다. `postgres`는 Postgres의 default role 이름이기도 하다. (`cat /etc/passwd | grep postgres`로 system account 확인 가능)

`systemctl`과 관련된 에러가 발생하면 [참고](https://devblogs.microsoft.com/commandline/systemd-support-is-now-available-in-wsl/).


## role 생성
- `sudo -u postgres createuser --interactive`: interactive한 방식으로 Postgres `role` 생성.
```
username@hostname:cwd$ sudo -u postgres createuser --interactive
Enter name of role to add: junseo
Shall the new role be a superuser? (y/n) y
```
- `psql` prompt에서 `ALTER USER rolename with encrypted password 'pwvalue';`: `rolename`의 password를 `pwvalue`로 변경. `psycopg2`를 이용해 flask와 연동할 때 password가 필요.


## 실행
- `sudo -u postgres psql`: `postgres@hostname`에서 `psql` 커맨드 실행. Postgres prompt에 접속.
- `\q, exit`: Postgres prompt에서 로그아웃.


## DB 생성
- `sudo -u postgres createdb dbname`
- `sudo -u rolename psql -d dbname`: `rolename`의 `role`로 `dbname`의 db에 접속.

특별한 옵션 없이 db에 접속을 시도하면(`psql`),  `role`과 같은 이름으로 된 db에 접속을 시도한다.
flask에서 사용할 `flask_db`라는 이름의 db를 생성한다. `GRANT ALL PRIVILEGES ON DATABASE flask_db TO rolename;`으로 `flask_db`에 대한 권한을 부여한다.


## 연동
- `pip install Flask psycopg2-binary`: `psycopg2` 설치
- `export POSTGRES_USERNAME='rolename'`, `export POSTGRES_PASSWORD='pwvalue'`
- `conn = psycopg2.connect(host='localhost', database='flask_db', user=os.environ['DB_USERNAME'], password=os.environ['DB_PASSWORD'])`
- `cur = conn.cursor()`
- `cur.execute('CREATE TABLE ~')`
- `cur.commit()`
- `cur.close(); conn.close()`


`os.environ[]`은 각각 `DB_USERNAME`과 `DB_PASSWORD`를 변수명으로 하는 환경 변수의 값을 가져온다.
`export` command를 이용해 환경 변수 세팅을 해준다.


## terminal
- prompt: `username@hostname:cwd$`
- `sudo su username`: `username`으로 switching account.
- `sudo -i`: superuser 권한이 부여된 새로운 shell에서 command 실행.
- `sudo -u username, --user=username`: 이후 command를 default target user(usually root)가 아닌 `username`가 실행하는 것으로 간주.


## 참고
- https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-20-04
- https://www.digitalocean.com/community/tutorials/how-to-use-a-postgresql-database-in-a-flask-application
- https://bbd531.tistory.com/entry/postgres-%EC%84%A4%EC%B9%98-%EB%B0%8F-%EC%B4%88%EA%B8%B0-%EC%84%A4%EC%A0%95
