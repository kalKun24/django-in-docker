## Django環境準備方法
### Step1. 環境変数ファイルを追加する
`dcoker-compose.yml`と同じ階層に、`.env`ファイルを作成する。
```
# [Dockerの設定]
# プロジェクト名
COMPOSE_PROJECT_NAME=
SRC_PATH=./src/ #プロジェクトを保存するディレクトリの指定(いじらなくてOK)

# [MYSQLの設定]
MYSQL_ROOT_PASSWORD= #ROOTパスワード
MYSQL_DATABASE= # DB名
MYSQL_USER= # DjangoからDBへのアクセスユーザ
MYSQL_PASSWORD= #アクセスユーザのパスワード
MYSQL_HOST=db

# [Djangoの設定]
# SECRET_KEYに任意の値を入力
# https://miniwebtool.com/ja/django-secret-key-generator/
DJANGO_SECRET_KEY=
DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
# デバッグ情報を出力するかどうか(開発環境はTrue)
DJANGO_DEBUG=True

```

主に編集する必要があるのは
- COMPOSE_PROJECT_NAME=
- MYSQL_ROOT_PASSWORD=
- MYSQL_DATABASE=
- MYSQL_USER=
- MYSQL_PASSWORD=
- DJANGO_SECRET_KEY=
の6項目である。

### Step2. docker-compose
1. `docker-compose.yml`と同じ階層にて、`docker-compose up -d`を叩く。
2. `docker ps`で`${COMPOSE_PROJECT_NAME}_web`, `${COMPOSE_PROJECT_NAME}_mysql`が立ち上がっていることを確認する。
3. `docker exec -it ${COMPOSE_PROJECT_NAME}_web /bin/bash`でdjango用コンテナに入る。
4. `django-admin startproject ${COMPOSE_PROJECT_NAME}`で、Djangoプロジェクトを生成する。
5. `src`配下にプロジェクトディレクトリが生成されていることを確認。

### Step3. settings.pyの編集
1. osライブラリをimportする。
```
import os
```

2. 基本情報の編集
```
SECRET_KEY = os.environ.get("DJANGO_SECRET_KEY")

DEBUG = os.environ.get("DJANGO_DEBUG")

ALLOWED_HOSTS = os.environ.get("DJANGO_ALLOWED_HOSTS").split(" ")
```

3. データベースの編集
```
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.mysql",
        # コンテナ内の環境変数をDATABASESのパラメータに反映
        "NAME": os.environ.get("MYSQL_DATABASE"),
        "USER": os.environ.get("MYSQL_USER"),
        "PASSWORD": os.environ.get("MYSQL_PASSWORD"),
        "HOST": os.environ.get("MYSQL_HOST"),
        "PORT": 3306,
        "OPTIONS": {
            "init_command": "SET sql_mode='STRICT_TRANS_TABLES'",
        },
        "TEST":{
            "NAME": "testSecuriTeachDB"
        }
    }
}
```

4. 言語とタイムゾーンの編集
```
LANGUAGE_CODE = 'ja'

TIME_ZONE = 'Asia/Tokyo'

USE_I18N = True

USE_TZ = True
```