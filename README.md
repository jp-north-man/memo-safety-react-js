## Javascript メモ
JavaScriptでREST APIを構築・利用する際の典型的な脆弱性とそれらの対策について<br>
### 1.不適切なエンドポイントの公開
エンドポイントが不適切に公開されている場合、不正なアクセスが許可される可能性があります。
```js
app.get('/admin/users', function(req, res) {
    // 全てのユーザーの情報を返す
});
```
対策: 適切な認証・認可を行い、権限のないユーザーがアクセスできないようにします。<br><br>

### 2.クロスサイトスクリプティング (XSS)
JavaScriptでAPIのレスポンスとして返されるデータが、クライアントサイドで安全でない方法で描画されるとXSSのリスクが生じます。
```js
let userData = JSON.parse(response);
document.getElementById('user').innerHTML = userData.name;
```
対策: データを安全にエスケープしてから表示します。また、.textContentを使用すると、HTMLとして解釈されないため、安全です。<br><br>

### 3.クロスサイトリクエストフォージェリ (CSRF)
APIを不正に呼び出されるリスクがあります。<br>
対策: CSRFトークンを使用して、APIのリクエストが正当なものか確認します。<br><br>

### 4.不適切な入力バリデーション
入力データがAPIに適切に検証されない場合、SQLインジェクションやその他の攻撃が可能になります。
```js
let query = `SELECT * FROM users WHERE name = ${req.body.name}`;
```
対策: パラメータ化クエリを使用し、またはORMツールを使用してデータアクセスを行います。<br><br>

### 5.不適切なエラーハンドリング
詳細なエラーメッセージを公開することは、攻撃者にシステムの情報を提供する可能性があります。
```js
app.get('/user', function(req, res) {
    // ...
    if (error) {
        res.send(`Error: ${error.message}`);
    }
});
```
対策: 一般的なエラーメッセージを返すようにし、詳細な情報はロギングします。<br><br>

### 6.不適切なCORS設定
全てのオリジンからのAPIアクセスを許可するような設定は、セキュリティのリスクとなります。
```js
app.use(function(req, res, next) {
  res.header("Access-Control-Allow-Origin", "*");
  next();
});
```
対策: 必要なオリジンのみを許可するようにCORSポリシーを設定します。<br><br>