# 問② 手順書
## Apache の DocumentRoot を `/home/saba` に変更し、`http://グローバルIP/miso/hoge.html` を表示する

## 1. 目的
Apache の DocumentRoot を `/home/saba` に変更したうえで、以下の URL にアクセスした際にコンテンツが表示されるようにする。

```text
http://グローバルIP/miso/hoge.html
```

このとき、実際のファイル配置は次のようになる。

```text
DocumentRoot: /home/saba
URL:          /miso/hoge.html
実ファイル:   /home/saba/miso/hoge.html
```

---

## 2. 前提
- OS: Amazon Linux 2023
- Apache(httpd) がインストール済みであること
- EC2 のセキュリティグループで HTTP(80) が許可されていること

---

## 3. Apache 設定ファイルを修正する

設定ファイルを開く。

```bash
sudo vi /etc/httpd/conf/httpd.conf
```

### 修正内容
`DocumentRoot` を `/home/saba` に変更し、`/home/saba` を公開できるように `<Directory>` 設定を追加または修正する。

```apache
DocumentRoot "/home/saba"

<Directory "/home/saba">
    AllowOverride None
    Require all granted
</Directory>
```

### 注意点
以下のように **終了タグから書いてしまうとエラー** になる。

```apache
</Directory "/home/saba">
```

正しくは次のように **開始タグ** で書く。

```apache
<Directory "/home/saba">
```

また、最後に必ず閉じタグを書く。

```apache
</Directory>
```

---

## 4. Apache の構文チェックと再起動

まず設定ファイルの構文チェックを行う。

```bash
sudo apachectl configtest
```

正常なら次のように表示される。

```text
Syntax OK
```

その後、Apache を再起動する。

```bash
sudo systemctl restart httpd
sudo systemctl status httpd
```

---

## 5. 公開用ディレクトリと HTML ファイルを作成する

`/home/saba` 配下に `miso` ディレクトリを作成する。

```bash
sudo mkdir -p /home/saba/miso
```

`hoge.html` を作成する。

```bash
sudo vi /home/saba/miso/hoge.html
```

`i` を押して入力モードにし、以下を記載する。

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>hoge</title>
</head>
<body>
  <h1>miso/hoge.html が表示されました</h1>
</body>
</html>
```

保存して終了する。

- `Esc`
- `:wq`
- `Enter`

---

## 6. 権限を設定する

Apache がファイルを参照できるように、権限を設定する。

```bash
sudo chmod 755 /home/saba
sudo chmod 755 /home/saba/miso
sudo chmod 644 /home/saba/miso/hoge.html
```

---

## 7. 動作確認

### 7-1. サーバー内で確認

```bash
curl http://localhost/miso/hoge.html
```

HTML が表示されれば Apache 側は正常。

### 7-2. ブラウザで確認

以下へアクセスする。

```text
http://グローバルIP/miso/hoge.html
```

---

## 8. エラー時の確認ポイント

### Apache の設定エラー確認

```bash
sudo apachectl configtest
sudo systemctl status httpd
sudo journalctl -xeu httpd.service
```

### 設定ファイルの該当箇所確認

```bash
sudo sed -n '124,136p' /etc/httpd/conf/httpd.conf
```

### HTML ファイルの存在確認

```bash
ls -l /home/saba/miso/hoge.html
```

---

## 9. まとめ
今回の設定では、`DocumentRoot` を `/home/saba` に変更するため、URL `/miso/hoge.html` に対応する実ファイルは `/home/saba/miso/hoge.html` となる。

つまり、

```text
http://グローバルIP/miso/hoge.html
```

を表示するには、以下が必要である。

- Apache の `DocumentRoot` を `/home/saba` に変更
- `<Directory "/home/saba">` に `Require all granted` を設定
- `/home/saba/miso/hoge.html` を作成
- Apache を再起動
