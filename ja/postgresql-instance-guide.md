## Database > PostgreSQL Instance > 使用ガイド

`このガイドは13バージョンを基準に作成されました。`  
`別のバージョンを使用している場合は、バージョンに合わせて変更してください。`

## PostgreSQL開始/停止方法

```
#postgresqlサービス開始
shell> systemctl start postgresql-13

#postgresqlサービス中止
shell> systemctl stop postgresql-13

#postgresqlサービス再起動
shell> systemctl restart postgresql-13
```

## PostgreSQL接続

イメージ作成後、最初は下記のように接続します。
<br>
```
#postgresにアカウント切り替え後、接続
shell> su - postgres
shell> psql
```

## PostgreSQLインスタンス作成後、初期設定

### 1\. ポート\(port\)変更

提供されるイメージポートはPostgreSQL基本ポート5432です。セキュリティ上、ポートの変更を推奨します。
<br>
```
shell> vi /var/lib/pgsql/13/data/postgresql.conf


#postgresql.confファイルに使用するポートを明記します。

port =使用するポート名


#viエディタ保存


#postgresqlサービス再起動

shell> systemctl restart postgresql-13


#変更されたポートに下記のように接続

shell> psql -p[変更されたポート番号]
```

### 2\. サーバーログタイムゾーン変更

サーバーログに記録される基本時間帯がUTCに設定されています。SYSTEMローカル時間と同じタイムゾーンに変更することを推奨します。
<br>
```
shell> vi /var/lib/pgsql/13/data/postgresql.conf


#postgresql.confファイルに使用するタイムゾーンを明記します。

log_timezone =使用するタイムゾーン


#viエディタ保存


#postgresqlサービス再起動

shell> systemctl restart postgresql-13


#postgresql接続

shell> psql


#変更した設定を確認

postgres=# SHOW log_timezone;
```

### 3\. publicスキーマ権限の削除

基本的にすべてのユーザーにpublicスキーマに対するCREATEおよびUSAGE権限を付与しているため、データベースに接続できるユーザーはpublicスキーマからオブジェクトを作成できます。すべてのユーザーがpublicスキーマからオブジェクトを作成できないように権限を削除すことを推奨します。
<br>
```
#postgresql接続

shell> psql


#権限削除コマンド実行

postgres=# REVOKE CREATE ON SCHEMA public FROM PUBLIC;
```

### 4\. 遠隔接続許可

ローカルホスト以外の接続を許可するにはlisten_addresses変数とクライアント認証設定ファイルを変更する必要があります。
<br>
```
shell> vi /var/lib/pgsql/13/data/postgresql.conf


#postgresql.confファイルに許可するアドレスを明記します。
#IPv4アドレスを全て許可する場合0.0.0.0
#IPv6アドレスを全て許可する場合::
#すべてのアドレスを許可する場合 *

listen_addresses =許可するアドレス


#viエディタ保存


shell> vi /var/lib/pgsql/13/data/pg_hba.conf


#IPアドレス形式ごとにクライアント認証制御
#古いクライアントライブラリはscram-sha-256方式がサポートされていないため、md5に変更必要

# TYPE  DATABASE        USER            ADDRESS                 METHOD
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
host    許可DB           許可ユーザー         許可アドレス                  scram-sha-256
# IPv6 local connections:
host    all             all             ::1/128                 scram-sha-256
host    許可DB           許可ユーザー         許可アドレス                  scram-sha-256


#postgresqlサービス再起動

shell> systemctl restart postgresql-13
```

## PostgreSQLディレクトリ説明

PostgreSQLディレクトリおよびファイルの説明は下記のとおりです。

| 名前 | 説明 |
| --- | --- |
| postgresql.cnf | /var/lib/pgsql/{version}/data/postgresql.cnf |
| initdb.log | PostgreSQLデータベースクラスター作成log - /var/lib/pgsql/{version}/initdb.log |
| DATADIR | PostgreSQLデータファイルパス - /var/lib/pgsql/{version}/data/ |
| LOG | PostgreSQL logファイルパス - /var/lib/pgsql/{version}/data/log/\*.log |
