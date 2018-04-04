mattermost-docker
============================================================

初期設定
------------------------------------------------------------

### HTTPS版

1. 証明書ディレクトリを作成

  ```
  mkdir -p /opt/docker/mattermost/volumes/web/cert
  cd /opt/docker/mattermost/volumes/web/cert
  ```

2. 鍵の作成

  ```
  openssl genrsa -aes128 2048 > localhost.key
  ```

  ```
  Generating RSA private key, 2048 bit long modulus
  ....................+++
  ................................+++
  e is 65537 (0x10001)
  Enter pass phrase: <パスフレーズ>
  Verifying - Enter pass phrase: <パスフレーズ>
  ```

3. 証明書の作成

  ```
  openssl req -utf8 -new -key localhost.key -x509 -days 3650 -out cert.pem -set_serial 0
  ```

  ```
  Enter pass phrase for localhost.key: <パスフレーズ>
  You are about to be asked to enter information that will be incorporated
  into your certificate request.
  What you are about to enter is what is called a Distinguished Name or a DN.
  There are quite a few fields but you can leave some blank
  For some fields there will be a default value,
  If you enter '.', the field will be left blank.
  -----
  Country Name (2 letter code) [AU]:JP
  State or Province Name (full name) [Some-State]: <都道府県名>
  Locality Name (eg, city) []: <都市名>
  Organization Name (eg, company) [Internet Widgits Pty Ltd]: <会社名/組織名>
  Organizational Unit Name (eg, section) []: <部門名>
  Common Name (e.g. server FQDN or YOUR name) []: <FQDN>
  Email Address []: <管理者メールアドレス>
  ```

4. 秘密鍵からパスフレーズの削除

  ```
  openssl rsa -in localhost.key -out key-no-password.pem
  ```

  ```
  Enter pass phrase for localhost.key: <パスフレーズ>
  writing RSA key
  ```

5. imageを作成する

    ```
    cd (mattermost-dockerのパス)
    docker-compose build
    ```

### HTTP版

1. 80番待受を有効化し、443待受を無効化する

  ```
  sed -i 's/###- "80:80"/- "80:80"/' docker-compose.yml
  sed -i 's/- "443:443"/###- "443:443"/' docker-compose.yml
  ```

2. imageを作成する

    ```
    docker-compose build
    ```

systemdによる自動起動設定
------------------------------------------------------------
host OSにsystemdの自動起動設定を行う
(ansibleのdocker imageが必要)

1. host OSにログインする

2. dockerからansibleの設定を行う

  ``` shell
  docker run --rm -it -v $(pwd)/systemd:/playbook hidepin/ansible ansible-playbook -i "(host OSのIPアドレス)," systemd.yml
  ```

参考
------------------------------------------------------------

### original

https://github.com/mattermost/mattermost-docker
