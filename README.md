## 概要
react,javascriptの脆弱性メモです。<br>

## Javascript
JavaScriptでREST APIを構築・利用する際の典型的な脆弱性とそれらの対策について<br>
### 1.不適切なエンドポイントの公開
エンドポイントが不適切に公開されている場合、不正なアクセスが許可される可能性があります。
```js
app.get('/admin/users', function(req, res) {
});
```
対策: 適切な認証・認可を行い、権限のないユーザーがアクセスできないようにします。<br><br>
例:JWTを検証するミドルウェア実装するなど
```js
const express = require('express');
const jwt = require('jsonwebtoken');
const expressJwt = require('express-jwt');

const app = express();
const SECRET_KEY = 'my-secret-key';
const jwtMiddleware = expressJwt({ secret: SECRET_KEY, algorithms: ['HS256'] });

app.get('/admin/users', jwtMiddleware, function(req, res) {
    if (!req.user.admin) {
        return res.status(403).send('だめ');
    }
});
```
<br><br>


### 2.クロスサイトスクリプティング (XSS)
JavaScriptでAPIのレスポンスとして返されるデータが、クライアントサイドで安全でない方法で描画されるとXSSのリスクが生じます。
```js
let userData = JSON.parse(response);
document.getElementById('user').innerHTML = userData.name;
```
対策: .innerHTMLを使用する代わりに.textContentを使用することは、テキストデータを安全にDOMに挿入します。指定されたテキストをそのままテキストとして描画します。HTMLやJavaScriptとして解釈されません。<br>
例:
```js
let userData = JSON.parse(response);
document.getElementById('user').textContent = userData.name;
```
<br><br>

### 3.不適切なエラーハンドリング
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
const logger = require('winston');
app.get('/user', function(req, res) {
    if (error) {
        // 詳細なエラー情報をロギングします
        logger.error(`Error fetching user: ${error.message}`);
        // 一般的なエラーメッセージをクライアントに返します
        res.status(500).send('Internal Server Error');
    }
});
```
<br><br>

### 4.不適切なCORS設定
全てのオリジンからのAPIアクセスを許可するような設定は、セキュリティのリスクとなります。
```js
app.use(function(req, res, next) {
  res.header("Access-Control-Allow-Origin", "*");
  next();
});
```
対策: 必要なオリジンのみを許可するようにCORSポリシーを設定します。<br><br>

### 5.不適切な状態管理
```jsx
localStorage.setItem('userToken', user.token);
```
対策: センシティブな情報（特に認証トークンなど）は安全な場所に保存します。例えば、トークンはHTTP Onlyのクッキーに保存すると良いです。<br><br>
例:
```js
res.cookie('userToken', userToken, { httpOnly: true, secure: true });
```
<br><br>

## React
### 1.クロスサイトスクリプティング (XSS)
ReactはJSXのコンテンツをデフォルトでエスケープしますが、dangerouslySetInnerHTMLの不適切な使用はXSS攻撃を許容する可能性があります。
```jsx
function UserProfile({ userData }) {
  return <div dangerouslySetInnerHTML={{ __html: userData.profile }} />;
}
```
対策: dangerouslySetInnerHTMLを使用する場合は、その内容が安全であることを確認するか、他の手段でデータを表示するようにします。<br><br>
例:JSX内に直接値を入れる
```jsx
function UserProfile({ userData }) {
  return <div>{userData.profile}</div>;
}

```
<br><br>

### 2.コンポーネントの隠蔽不足
認証や認可が必要なコンポーネントを、適切に隠蔽しないと情報が漏洩する可能性があります。
```jsx
function AdminPanel() {
}
```
対策: 認証や認可のロジックを適切に実装（useContextなどを使う）、不正なアクセスを防ぎます。<br><br>

### 3.クロスサイトリクエストフォージェリ (CSRF)
APIを不正に呼び出されるリスクがあります。<br>
対策: CSRFトークンを使用して、APIのリクエストが正当なものか確認します。<br><br>
例:
```js
const [csrfToken, setCsrfToken] = useState(null);

useEffect(() => {
  axios.get('/api/csrf-token').then(response => {
    setCsrfToken(response.data.token);
  });
}, []);

const handleAction = () => {
  axios.post('/api/action', {}, {
    headers: {
      'CSRF-Token': csrfToken
    }
  }).then(response => {
    console.log(response.data);
  }).catch(error => {
    console.error("Error:", error);
  });
};

```
<br><br>