<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [インベントリと変数、環境について](#インベントリと変数環境について)
    - [ホスト変数とグループ変数](#ホスト変数とグループ変数)
    - [Gitリポジトリ](#gitリポジトリ)
- [プレイブックとロールの説明](#プレイブックとロールの説明)
- [プレイブックの実行方法](#プレイブックの実行方法)
    - [変数 dns_entries について](#変数-dnsentries-について)
- [Ansible Towerへの設定例](#ansible-towerへの設定例)
- [Pre-commit using yamllint and ansible-lint](#pre-commit-using-yamllint-and-ansible-lint)

<!-- markdown-toc end -->



インベントリと変数、環境について
================

- "unbound"グループの直下に必要なだけサブグループを定義する。
  - ここでは"proj1"と"proj2"の2つのサブグループが定義されており、それぞれ2台と1台のホストがいる。
- "bastion"という名前のホストを用意する。
  - bastionではUnboundをサービスとしては動かさず、代わりにGitとPodmanを動かしテストに用いる。
  - ちょうど別にAnsible Towerがあるので、ここではそれにマップして代用している。
  - バリデーションにipv4フィルターを用いているので`pip install netaddr`が必要だが、Ansible Towerでは既にインストール済みである。
- いずれのホストもRHEL 8とする。

```
@all:
  |--@unbound:
  |  |--@proj1:
  |  |  |--unbound1.example.com
  |  |  |--unbound2.example.com
  |  |--@proj2:
  |  |  |--rhel8-sandbox.example.com
  |--@ungrouped:
  |  |--bastion (tower.example.com)
```

## ホスト変数とグループ変数

- unbound_listen_addresses
  - Unboundサーバがリスンするアドレスをリストで定義する。
  - unbound.confの`interface`に設定される。
- unbound_allowed_networks
  - Unboundサーバが再起問い合わせを受け付けるネットワークをCIDRのリストで定義する。
  - unbound.confの`access-control`に`allow`で設定される。
- unbound_default_nameservers
  - Unboundサーバが管理下にないドメインの問い合わせを転送するDNSサーバをリストで定義する。
  - unbound.confの`forward-addr`に設定される。
- unbound_managing_domains
  - このUnboundサーバで管理するドメインのリストを定義する。
  - unbound.confの`local-zone`に設定される。
- unbound_data.pull_url
  - unbound-dataリポジトリのpull可能なリポジトリのURL（通常はhttpsを使う）を指定する。
  - unboudnロールがデータファイルを取得するのに使う。
- unbound_data.push_url
  - "unbound-data"のpullだけでなくpushも可能なリポジトリのURL（通常はsshを使う）を指定する。
  - unbound_dataロールがデータファイルを取得・編集・保存するのに使う。
- unbound_data.version
  - unbound-dataリポジトリの使用するタグやコミットIDを指定する。

参考リンク:
- [hosts.yml](hosts.yml)
- [unbound.conf](https://nlnetlabs.nl/documentation/unbound/unbound.conf/)

## Gitリポジトリ

- [unbound-playbooks](https://github.com/onagano-rh/unbound-playbooks)
  - 本リポジトリ。インベントリ、各プレイブック、各ロールを含む
  - 使用するロールは全て本リポジトリのrolesディレクトリにあり、Ansible Galaxyは使っていない。
- [unbound-data](https://github.com/onagano-rh/unbound-data)
  - 管理するドメインごとにファイルに分けてデータを保存するリポジトリ。
  - Unboundサーバの /etc/unbound/conf.d/ にコピーしてそのまま読み込まれる。
  - unbound_dataロールが全体を生成・保存し、実行の度にコメントは除去され、行もソートされる。
    - 最初にファイルを用意するときを除いて、マニュアルでの編集は想定されていない。
  - 管理するドメインごとにconfファイルを用意する。
    - ファイル名は"<ドメイン名>.conf"にする。
    - ファイルの行頭に"local-zone: <ドメイン名> <タイプ>"を書いておく。
      - "<タイプ>"には"static"や"transparent"などと記述する。
      - 詳細は [unbound.conf(5)](https://nlnetlabs.nl/documentation/unbound/unbound.conf/#local-zone) を参照。
    - local-dataはプレイブック実行で追加できるが、ファイルとlocal-zone行の準備はマニュアルで行う。
  - ファイルの階層関係は持たせられない。
    - 例えば、"hoge.example.com.conf"ファイルが既にあるなら、"sub.hoge.example.com.conf"は用意せず、subのDNSエントリも"hoge.example.com.conf"ファイルに記述する。

参考リンク:
- [Unbound，知ってる？　この先10年を見据えたDNS](https://gihyo.jp/admin/feature/01/unbound/0004)



プレイブックとロールの説明
================

依存関係:

- [update_unbound_data.yml](update_unbound_data.yml)
  - [roles/unbound_data](roles/unbound_data)
    - [roles/git](roles/git)
    - [roles/podman](roles/podman)
- [deploy_unbound.yml](deploy_unbound.yml)
  - [roles/unbound](roles/unbound)
    - [roles/git](roles/git)

注記:

- ロールには meta/main.yml の dependencies を通して依存関係がある。
  - gitとpodmanのロールは汎用的になっており、特にgitロールは各プレイブックで間接的に使われる。

参考リンク:
- [Playbook Keywords](https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html)
- [Using Roles (タスクの実行順序)](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#using-roles)
- [16. ジョブテンプレート (Tower上で定義されるマジック変数の一覧)](https://docs.ansible.com/ansible-tower/3.6.2/html_ja/userguide/job_templates.html)
- [Delegation, Rolling Updates, and Local Actions](https://docs.ansible.com/ansible/latest/user_guide/playbooks_delegation.html)
- [Error Handling In Playbooks (force_handlers)](https://docs.ansible.com/ansible/latest/user_guide/playbooks_error_handling.html)
- [Creating Reusable Playbooks, Dynamic vs. Static](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse.html#dynamic-vs-static)

## プレイブック update_unbound_data.yml について

  - およそ以下のような処理を行う。
    - 後述する変数"dns_entries"を解析・バリデーションし、ドメイン名からそれを管理するグループを割り出す。
    - unbound-dataリポジトリを一時ディレクトリにクローンする。
    - テスト用にUnboundが入ったPodmanコンテナをビルドおよび起動する。
    - データファイルの行の更新・削除を実行する。
    - Podmanコンテナに対してdigコマンドでテストを行う。
    - 更新・削除があればunbound-dataリポジトリにpushする。
    - 一時ディレクトリやPodmanコンテナを削除する。
  - 作業は全てbastionホストで、ホスト数に関わらず一回だけ行われる。
    - ロールのインポート時に`delegate_to: bastion`および`run_once: true`を指定しているため。
    - `hosts: bastion`ではなく`host: all`としているのは、`--limit`（もしくは`-l`）オプションを活用するため。
      - `ansilbe-playbook`コマンドやAnsible Towerのジョブテンプレートで、リミットにより対象グループを制限できる方が直感的で使いやすい。
      - プレイブックの`hosts: `で指定したグループと`--limit`オプションで指定したグループには、共通部分がなければ何も実行されない。
      - `--limit`オプションの値はマジック変数ansible_limitで取得でき、バリデーションの際に使用している。
        - 例えば、`--limit proj1`と指定しているのに、dns_entriesでproj1では管理していないドメインが含まれていると、エラーにする。
      - ただし、`--limit`オプションには複数のグループは指定できない。
  - このプレイブックは、bastionでのGitとPodmanの操作およびunbound-dataのリポジトリにのみ関わり、unboundグループのサーバには何も行わない。
    - Gitの一時ディレクトリとして "/tmp/unbound_<NN>" を、Podmanの一時コンテナとして "unbound_<NN>" を用いる。
    - プレイブックが途中でエラーになった場合はそれらが残るので、マニュアルで削除する。
      - この削除処理はgitおよびpodmanの各ロールのハンドラーで行われる。
      - プレイブックに`force_handlers: true`と書くことでエラー時にもハンドラーを起動することはできるが、トラブルシュートのためにもこれはデフォルトのままfalseにしている。
    - ハンドラーは、実際にはロールのタスク内で毎回notifyされ、ハンドラーのタスクのwhenの条件を見て実際の処理を行うかスキップするかを判断している。
      - そのために`git_enable_remove_handler`や`podman_enable_remove_handler`といった変数をハンドラーごとに用意し、ロールの defaults/main.yml 内で適切なデフォルト値を設定している。

## プレイブック deploy_unbound.yml について

  - unboundグループのサーバに対して、unboundロールを用いてデータファイルを配置する。
    - `serial: 1`が指定されており、サーバ一台ずつに動作していく。
  - Unboundのサービスをゼロから構築せねばならなくなったときなどは、このプレイブックのみを用いてGitの特定バージョンから状態を復旧できる。
    - unbound-dataリポジトリにあるテスト済みのconfファイルを用いてシンプルに動作する。
    - 復旧時にはGitでpullさえできればよくpushは必要ないので、特にpull用のURLの変数を設けている。
      - httpsで自己署名証明書を使っている場合はgitロールで定義されている変数の設定`git_config_ssl_verify: false`を忘れないようにする。
  - /etc/resolv.conf の編集は行わない。



プレイブックの実行方法
================

特にカスタマイズが必要な変数は、毎回違う値が想定されているdns_entriesの他には、
unbound-dataリポジトリのURLのみである。
毎回`--extra-vars`（または`-e`）オプションで上書きするよりは、インベントリの
変数としてあらかじめ書き換えておく。

```
    unbound_data_push_url: git@gitlab.example.com:test-user/unbound-data.git
    unbound_data_pull_url: https://gitlab.example.com/test-user/unbound-data.git
```

Gitでpushする際のSSHの秘密鍵を、[roles/git/files](roles/git/files)
内に、`ansible-vault encrypt`した状態であらかじめコピーしておく。
ファイル名が"id_rsa"と違う場合は変数git_ssh_private_keyも上書きしておく。

```shell
$ cp ~/.ssh/id_ras roles/git/files/id_rsa
$ echo -n '<Vaultパスワード>' > /tmp/my-password.txt
$ ansible-vault encrypt --vault-password-file /tmp/my-password.txt roles/git/files/id_rsa
```

その他gitロールに関しては、コミット時のユーザ名とメールアドレス、
コミットログのメッセージのテンプレートなど、変えておいた方がよいものもある。

- [roles/git/defaults/main.yml](roles/git/defaults/main.yml)
- [roles/git/templates/commit-log-message.txt.j2](roles/git/templates/commit-log-message.txt.j2)

ロールが使用する変数については各プレイブック内で適切な値に記述済みである。
各ロール変数はロール名でプレフィクスをつけており、そのロールの defaults/main.yml
内に定義が容易に見つかる。

まず、後述するdns_entriesをtest-data.ymlに記述して、
それとVaultパスワードを指定してupdate_unbound_data.ymlを実行する。

```shell
$ vi test-data.yml
$ ansible-playbook update_unbound_data.yml \
  -e @test-data.yml \
  --vault-password-file /tmp/my-password.txt
```

必要に応じて、dns_entriesの内容と整合するリミットオプションを付けて対象を制限することができる。

```shell
$ ansible-playbook update_unbound_data.yml \
  -e @test-data.yml \
  --vault-password-file /tmp/my-password.txt -l proj1
```

deploy_unbound.ymlのプレイブックに関しては`git push`は行わないので
dns_entries変数やVaultパスワードの指定は要らない。

```shell
$ ansible-playbook deploy_unbound.yml
```
リミットオプションについてはupdate_unbound_data.ymlと同様である。

```shell
$ ansible-playbook deploy_unbound.yml -l proj1
```

なお、現時点のpodmanとunboundパッケージではrootユーザでないと正常にインストールや実行が
できなかったため、インベントリでは`ansible_user: root`としているが、RHELのアップデート
次第ではこの制約はなくなると考えられる。

## 変数 dns_entries について

以下のようなアイテムのリストとしてdns_entriesを定義する。

```yaml
dns_entries:
  - name: svr01.hoge.example.com
    address: 10.1.1.1
  - name: svr01.hoge.example.com
    address: 10.1.1.2
  - name: svr09.bar.example.com
    state: absent
```

- DNSエントリを追加したい場合は"name"と"address"を指定する。
  - `unbound-control local_data`コマンドによりデータが投入される。
  - 既にあるエントリを指定してもエラーにはならない（追加と更新に違いはない）。
- 同一ホストに対して複数のアドレスを定義する場合は、同じ"name"で"address"を変える。
- DNSエントリを削除したい場合は"name"と"state: absent"を指定する。
  - `unbound-control local_data_remove`コマンドによりデータが削除される。
  - 存在しないエントリに対して削除を実行してもエラーにはならない。
  - 複数のアドレスを持つエントリの場合、それらのアドレス全てが削除される。
- 削除と追加・更新の違いは"state"が"absent"かどうかによって判断しており、その他の指定は単に無視される。
  - リストの順番に対応する`unbound-control`コマンドが実行される。
  - 従って、複数アドレスの一部を削除したい場合は、まず削除しその後必要なアドレスを再登録する。
- 設定ファイルへの書き出しは`unbound-control list_local_data`コマンドの出力を整形して行っている。
  - 設定ファイルの書式チェックとして`unbound-checkconf`コマンドを使っている。

参考リンク:
- [unbound-control(8)](https://nlnetlabs.nl/documentation/unbound/unbound-control/)
- [unbound-checkconf(8)](https://nlnetlabs.nl/documentation/unbound/unbound-checkconf/)

## 未知のバグについて

site.ymlとして両プレイブックを連続実行するものも考えられる。

```shell
$ cat site.yml
---
- import_playbook: update_unbound_data.yml
- import_playbook: deploy_unbound.yml
```

ただし、不明な条件で、定義しているはずのunbound_data_pull_url変数が見つからないという
エラーが発生し、未知のバグの存在が疑われる。

```
TASK [git : Clone the repository to ./unbound-data.git] ***************************************************************
Saturday 21 March 2020  22:13:58 +0900 (0:00:00.014)       0:02:42.043 ********
ERROR! 'unbound_data_pull_url' is undefined
```

そのため、安全策でsite.ymlは実行せず、各プレイブックを分て実行することにする。
複数のプレイブックを連携させるには、Ansible Tower上ではワークフローを使うことになる。



Ansible Towerへの設定例
================

参考リンク:
- [jsmartin/tower_populator](https://github.com/jsmartin/tower_populator)
- [Ansible Tower CLI](https://tower-cli.readthedocs.io/en/latest/index.html)
- [AWX Command Line Interface](https://github.com/ansible/awx/tree/devel/awxkit/awxkit/cli/docs)
- [16.1. インベントリーのインポート](https://docs.ansible.com/ansible-tower/3.6.2/html_ja/administration//tower-manage.html#inventory-import)



Pre-commitの使い方
================

インストール方法:

    pip install pre-commit yamllint ansible-lint

もしくは"molecule"の依存性により:

    pip install molecule

設定:

    pre-commit sample-config > .pre-commit-config.yaml
    vi .pre-commit-config.yaml

- For yamllint
  - https://yamllint.readthedocs.io/en/stable/integration.html#integration-with-pre-commit
- For ansible-lint
  - https://github.com/ansible/ansible-lint#pre-commit-setup
  - [`entry: ansible-lint`を追加しないと動かない？](.pre-commit-config.yaml#L27)

マニュアルでのlintの起動方法:

    pre-commit run --all-files
    yamllint .
    ansible-lint

コミット時に実行されるのはコミットしようとするファイルのみなので、
任意のファイルをチェックしたいなら上記のように実行する。

コミット時に自動実行されるよう.git/hooks/pre-commitをインストール:

    pre-commit install

自動実行をやめる場合:

    pre-commit uninstall
