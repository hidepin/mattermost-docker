mattermost-docker
============================================================

初期設定
============================================================

1. imageを作成する

    ```
    docker-compose build
    ```

systemdによる自動起動設定
============================================================
host OSにsystemdの自動起動設定を行う
(ansibleのdocker imageが必要)

1. host OSにログインする

2. dockerからansibleの設定を行う

  ``` shell
  docker run --rm -it -v $(pwd)/systemd:/playbook hidepin/ansible ansible-playbook -i "(host OSのIPアドレス)," systemd.yml
  ```
