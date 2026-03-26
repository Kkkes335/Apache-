# 問④ 手順書
## 自分のサーバーからのアクセスは許可し、相手のサーバーからのアクセスは拒否する設定

## 1. 目的
Apache で `/apple` 配下にアクセス制御を設定し、以下の要件を満たす。

- 自分のサーバーからのアクセスは許可する（HTTP 200）
- 相手のサーバーからのアクセスは拒否する（HTTP 403）

確認イメージ:

```text
curl -LI http://自分プライベートIP/apple/xxx   → 200
curl -LI http://相手のプライベートIP/apple/xxx → 403
```

## 2. 前提
- OS: Amazon Linux 2023
- Apache(httpd) がインストール済みで起動していること
- `/apple/xxx` に対応する公開ファイルを配置すること
- 相手サーバーのプライベートIPが分かっていること

本手順では、Apache 2.4 の `Require` を利用してアクセス制御を行う。

## 3. 公開用ディレクトリとファイルを作成する
まず `/apple` 配下の公開ディレクトリと確認用ファイルを作成する。

```bash
sudo mkdir -p /var/www/html/apple
sudo vi /var/www/html/apple/xxx
```

`vi` で開いたら `i` を押して入力モードにし、たとえば以下の内容を入力する。

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>apple test</title>
</head>
<body>
  <h1>apple/xxx の確認ページです</h1>
</body>
</html>
```

保存して終了する。

- `Esc`
- `:wq`
- `Enter`

権限も設定する。

```bash
sudo chmod 755 /var/www/html
sudo chmod 755 /var/www/html/apple
sudo chmod 644 /var/www/html/apple/xxx
```

## 4. Apache にアクセス制御を設定する
Apache 設定ファイルを開く。

```bash
sudo vi /etc/httpd/conf/httpd.conf
```

末尾付近に次の設定を追加する。

```apache
<Directory "/var/www/html/apple">
    <RequireAll>
        Require all granted
        Require not ip 相手のプライベートIP
    </RequireAll>
</Directory>
```

### 設定例
相手サーバーのプライベートIPが `172.31.10.20` の場合:

```apache
<Directory "/var/www/html/apple">
    <RequireAll>
        Require all granted
        Require not ip 172.31.10.20
    </RequireAll>
</Directory>
```

### 設定の意味
- `Require all granted`
  - まず全体を許可する
- `Require not ip 相手のプライベートIP`
  - そのうえで、相手サーバーのIPだけ拒否する
- `<RequireAll>`
  - 中の条件をまとめて評価する

この設定により、相手サーバーから `/apple` 配下へアクセスした場合は **403 Forbidden** になる。

## 5. Apache の構文チェックと再起動
設定反映前に構文チェックを行う。

```bash
sudo apachectl configtest
```

正常であれば次のように表示される。

```text
Syntax OK
```

問題がなければ Apache を再起動する。

```bash
sudo systemctl restart httpd
sudo systemctl status httpd
```

## 6. 動作確認

### 6-1. 自分のサーバーから確認
自分のサーバー上で実行する。

```bash
curl -LI http://自分プライベートIP/apple/xxx
```

期待結果:

```text
HTTP/1.1 200 OK
```

### 6-2. 相手のサーバーから確認
相手サーバー上で、自分サーバーのプライベートIP宛に実行する。

```bash
curl -LI http://自分プライベートIP/apple/xxx
```

期待結果:

```text
HTTP/1.1 403 Forbidden
```

## 7. トラブルシュート

### Apache の設定エラー確認
```bash
sudo apachectl configtest
sudo systemctl status httpd
sudo journalctl -xeu httpd.service
```

### 設定ファイルの末尾確認
```bash
sudo tail -n 20 /etc/httpd/conf/httpd.conf
```

### 公開ファイルの存在確認
```bash
ls -l /var/www/html/apple/xxx
```

### 自分サーバー内で localhost 確認
```bash
curl -LI http://localhost/apple/xxx
```

## 8. 補足
今回の設定は、`/apple` 配下に対して **相手サーバーのプライベートIPだけを拒否**する方法である。

そのため、要件である以下を満たせる。

```text
自分のサーバーからのアクセス → 200
相手のサーバーからのアクセス → 403
```

相手サーバーのIPが変わる場合は、`Require not ip` の値も修正すること。

## 9. まとめ
本設定では、Apache の `<Directory "/var/www/html/apple">` に対して IP ベースのアクセス制御を設定することで、相手サーバーからのアクセスのみを拒否し、自分のサーバーからのアクセスは許可する。

設定の要点は以下の通り。

```apache
<Directory "/var/www/html/apple">
    <RequireAll>
        Require all granted
        Require not ip 相手のプライベートIP
    </RequireAll>
</Directory>
```

この設定後、`curl -LI` による確認で 200 と 403 の差を確認する。
