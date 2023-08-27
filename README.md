## 概要
react,javascriptの脆弱性メモ（自分用）です。<br>

## Javascript
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
対策: データを安全にエスケープしてから表示します。また、.textContentを使用すると、HTMLとして解釈されないため、安全です。<br>
例:
```js
let userData = JSON.parse(response);
document.getElementById('user').textContent = userData.name;
```
<br><br>

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
対策: 一般的なエラーメッセージを返すようにし、詳細な情報はロギングします。<br>
例:
```js
const logger = require('winston');  // 例としてwinstonロギングライブラリを使用
app.get('/user', function(req, res) {
    // ...
    if (error) {
        // 詳細なエラー情報をロギング
        logger.error(`Error fetching user: ${error.message}`);
        // 一般的なエラーメッセージをクライアントに返す
        res.status(500).send('Internal Server Error');
    }
});
```
<br><br>

### 6.不適切なCORS設定
全てのオリジンからのAPIアクセスを許可するような設定は、セキュリティのリスクとなります。
```js
app.use(function(req, res, next) {
  res.header("Access-Control-Allow-Origin", "*");
  next();
});
```
対策: 必要なオリジンのみを許可するようにCORSポリシーを設定します。<br><br>

## React
### 1.クロスサイトスクリプティング (XSS)
ReactはJSXのコンテンツをデフォルトでエスケープしますが、dangerouslySetInnerHTMLの不適切な使用はXSS攻撃を許容する可能性があります。
```jsx
function UserProfile({ userData }) {
  return <div dangerouslySetInnerHTML={{ __html: userData.profile }} />;
}
```
対策: dangerouslySetInnerHTMLを使用する場合は、その内容が安全であることを確認するか、他の手段でデータを表示するようにします。<br><br>

### 2.不適切な状態管理
```jsx
localStorage.setItem('userToken', user.token);
```
対策: センシティブな情報は安全な場所に保存します。例えば、トークンはHTTP Onlyのクッキーに保存すると良いでしょう。<br><br>

### 3.コンポーネントの隠蔽不足
認証や認可が必要なコンポーネントを、適切に隠蔽しないと情報が漏洩する可能性があります。
```jsx
function AdminPanel() {
  // ...
}
```
対策: 認証や認可のロジックを適切に実装し、不正なアクセスを防ぎます。<br><br>

### 4.不適切なエラーハンドリング
エラーメッセージをユーザーに公開すると、アプリケーションの詳細や構造が露呈する恐れがあります。
```jsx
function FetchData() {
  const [error, setError] = useState(null);
  // ...
  if (error) {
    return <div>Error: {error.message}</div>;
  }
}
```
対策: エラーメッセージはロギングし、ユーザーには一般的なエラーメッセージのみを表示します。<br><br>
