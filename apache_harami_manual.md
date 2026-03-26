# Amazon Linux 2023 / Apache で `http://グローバルIP/yakiniku/harami.html` を表示する手順書

## 1. 概要

本手順書は、Amazon Linux 2023 上に Apache（httpd）を構築し、以下の URL で HTML コンテンツを表示できるようにするための手順です。

```text
http://グローバルIP/yakiniku/harami.html
```

今回のポイントは、Apache の `DocumentRoot` をファイルに設定するのではなく、通常の公開ディレクトリ配下に HTML ファイルを配置することです。

- `DocumentRoot` : `/var/www/html`
- 配置する実ファイル : `/var/www/html/yakiniku/harami.html`

---

## 2. 前提条件

- EC2 インスタンスが Amazon Linux 2023 で作成されていること
- SSH 接続できること
- セキュリティグループで以下が許可されていること
  - SSH（TCP 22）: 自分の接続元IP
  - HTTP（TCP 80）: 必要に応じて `0.0.0.0/0`

---

## 3. EC2 へ接続

以下のコマンドで EC2 に接続します。

```bash
ssh -i 鍵ファイル.pem ec2-user@グローバルIP
```

---

## 4. Apache のインストール

パッケージを更新し、Apache（httpd）をインストールします。

```bash
sudo dnf upgrade -y
sudo dnf install -y httpd
```

---

## 5. Apache の起動と自動起動設定

Apache を起動し、OS 再起動後も自動起動するように設定します。

```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

状態確認:

```bash
sudo systemctl status httpd
```

---

## 6. 公開ディレクトリの作成

今回表示したい URL は `/yakiniku/harami.html` なので、`DocumentRoot` 配下に `yakiniku` ディレクトリを作成します。

```bash
sudo mkdir -p /var/www/html/yakiniku
```

---

## 7. `vi` で `harami.html` を作成

以下のコマンドで HTML ファイルを開きます。

```bash
sudo vi /var/www/html/yakiniku/harami.html
```

### `vi` の操作手順

1. `i` キーを押して入力モードにする
2. 下記の HTML を入力する
3. `Esc` キーを押す
4. `:wq` と入力して Enter を押す

入力内容:

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Harami Page</title>
</head>
<body>
  <h1>harami.html が表示されました</h1>
  <p>Apache on Amazon Linux で公開できています。</p>
</body>
</html>
```

---

## 8. パーミッション設定

必要に応じて、ディレクトリとファイルの権限を設定します。

```bash
sudo chmod 755 /var/www/html
sudo chmod 755 /var/www/html/yakiniku
sudo chmod 644 /var/www/html/yakiniku/harami.html
```

---

## 9. サーバー内での確認

まずは EC2 サーバー内で正しく表示できるか確認します。

```bash
curl http://localhost/yakiniku/harami.html
```

HTML の内容が表示されれば、Apache 側の設定は正常です。

---

## 10. ブラウザでの確認

ローカル PC のブラウザから以下へアクセスします。

```text
http://グローバルIP/yakiniku/harami.html
```

`harami.html が表示されました` と出れば成功です。

---

## 11. うまく表示されない場合の確認項目

### 11.1 Apache が起動していない

```bash
sudo systemctl status httpd
sudo journalctl -u httpd -e
```

### 11.2 ファイルが正しい場所にない

```bash
ls -l /var/www/html/yakiniku/harami.html
```

### 11.3 セキュリティグループで HTTP(80) が許可されていない

AWS マネジメントコンソールで、EC2 のセキュリティグループを確認してください。

- HTTP / TCP / 80 が許可されていること

### 11.4 URL を間違えている

正しい URL は以下です。

```text
http://グローバルIP/yakiniku/harami.html
```

`/harami.html` だけではなく、`/yakiniku/harami.html` まで含める必要があります。

---

## 12. 最短手順まとめ

```bash
sudo dnf upgrade -y
sudo dnf install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
sudo mkdir -p /var/www/html/yakiniku
sudo vi /var/www/html/yakiniku/harami.html
sudo chmod 755 /var/www/html
sudo chmod 755 /var/www/html/yakiniku
sudo chmod 644 /var/www/html/yakiniku/harami.html
curl http://localhost/yakiniku/harami.html
```

---

## 13. 補足

- `DocumentRoot` は通常 `/var/www/html` のままでよいです。
- 今回のように URL に `/yakiniku/harami.html` を含めたい場合は、`DocumentRoot` 配下に `yakiniku/harami.html` を作成します。
- `DocumentRoot` を `/yakiniku/harami.html` のようなファイルに設定することはしません。
