# 問③ 手順書
## `http://グローバルIP/sa-mon.html` へアクセスしたら、ペアの `aburi-sa-mon.html` へリダイレクトさせる

## 1. 目的
自分の Apache サーバーで以下の URL にアクセスしたとき、ペアのサーバー上のページへリダイレクトさせる。

```text
http://自分のグローバルIP/sa-mon.html
```

アクセス後、以下へ遷移させる。

```text
http://ペアのグローバルIP/aburi-sa-mon.html
```

例:

```text
http://111.111.111.111/sa-mon.html
↓
http://222.222.222.222/aburi-sa-mon.html
```

---

## 2. 前提
- OS: Amazon Linux 2023
- Apache(httpd) がインストール済みで起動していること
- 自分の EC2 のセキュリティグループで HTTP(80) が許可されていること
- ペア側サーバーで `http://ペアのグローバルIP/aburi-sa-mon.html` が実際に表示できること

---

## 3. リダイレクトの考え方
今回は自分のサーバー上で `/sa-mon.html` へのアクセスを受けたら、別の URL へ転送する。

Apache では `Redirect` ディレクティブを使うと簡単に設定できる。

設定の考え方:

```text
受けるURL:   /sa-mon.html
転送先URL:   http://ペアのグローバルIP/aburi-sa-mon.html
```

---

## 4. Apache 設定ファイルを開く

```bash
sudo vi /etc/httpd/conf/httpd.conf
```

---

## 5. リダイレクト設定を追加する
設定ファイルの末尾付近に、次の 1 行を追加する。

```apache
Redirect /sa-mon.html http://ペアのグローバルIP/aburi-sa-mon.html
```

### 記入例
ペアのグローバルIPが `222.222.222.222` の場合:

```apache
Redirect /sa-mon.html http://222.222.222.222/aburi-sa-mon.html
```

### 補足
- 左側 `/sa-mon.html` は、自分のサーバーで受けるパス
- 右側 `http://222.222.222.222/aburi-sa-mon.html` は、飛ばしたい先の URL

通常は `sa-mon.html` の実ファイルを自分のサーバー上に作成する必要はない。
`Redirect` 設定が先に働くため、アクセス時に転送される。

---

## 6. 構文チェックを行う
設定を保存したら、Apache の構文チェックを行う。

```bash
sudo apachectl configtest
```

正常なら次のように表示される。

```text
Syntax OK
```

---

## 7. Apache を再起動する

```bash
sudo systemctl restart httpd
sudo systemctl status httpd
```

`active (running)` になっていればよい。

---

## 8. 動作確認

### 8-1. サーバー内で確認する
HTTP ヘッダだけ確認する場合:

```bash
curl -I http://localhost/sa-mon.html
```

成功時の例:

```text
HTTP/1.1 302 Found
Location: http://222.222.222.222/aburi-sa-mon.html
```

`Location:` に転送先 URL が表示されれば成功。

### 8-2. ブラウザで確認する
ブラウザで以下へアクセスする。

```text
http://自分のグローバルIP/sa-mon.html
```

ペアの以下の URL へ遷移すれば成功。

```text
http://ペアのグローバルIP/aburi-sa-mon.html
```

---

## 9. うまくいかないときの確認ポイント

### 9-1. Apache の設定エラー確認

```bash
sudo apachectl configtest
sudo systemctl status httpd
sudo journalctl -xeu httpd.service
```

### 9-2. 設定が正しく入っているか確認

```bash
grep -n "Redirect /sa-mon.html" /etc/httpd/conf/httpd.conf
```

### 9-3. ペア側 URL が本当に表示できるか確認
自分のサーバーから転送先へ直接アクセスして確認する。

```bash
curl -I http://ペアのグローバルIP/aburi-sa-mon.html
```

ここで相手側が `404` や接続不可の場合、リダイレクト設定自体は正しくても、転送先ページが未作成の可能性がある。

### 9-4. 自分のセキュリティグループを確認
自分の EC2 に対して HTTP(80) が許可されていないと、ブラウザでアクセスしても確認できない。

---

## 10. 恒久的リダイレクトにしたい場合
今回の `Redirect` は通常 302 の一時的リダイレクトとして動作する。
恒久的に転送したい場合は、次のように書く。

```apache
Redirect permanent /sa-mon.html http://222.222.222.222/aburi-sa-mon.html
```

ただし、研修や確認用途では通常の `Redirect` で十分。

---

## 11. まとめ
今回の問③では、自分の Apache に次の設定を入れることで、`/sa-mon.html` へのアクセスをペアのページへ転送できる。

```apache
Redirect /sa-mon.html http://ペアのグローバルIP/aburi-sa-mon.html
```

作業の流れは以下の通り。

1. `/etc/httpd/conf/httpd.conf` を開く
2. `Redirect /sa-mon.html http://ペアのグローバルIP/aburi-sa-mon.html` を追記する
3. `sudo apachectl configtest` で構文確認する
4. `sudo systemctl restart httpd` で再起動する
5. ブラウザまたは `curl -I` で動作確認する
