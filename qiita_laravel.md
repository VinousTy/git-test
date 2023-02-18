## はじめに

Laravel のルーティングファイルについて、何気なく触っていたので備忘録として残します。
※Laravel8 を使用

## 対象者

この記事は下記のような人を対象にしています。

- 駆け出しエンジニア
- プログラミング初学者

## ルーティングファイルの種類

- web.php
- api.php

## 違いについて

#### web.php

デフォルトで CSRF 保護の機能が有効になっているため、外部から POST することができない。

#### api.php

CSRF 保護が有効になっていないため外部から POST ができる。

## 詳細

kernel.php に web.php と api.php のそれぞれの設定がミドルウェアとして組み込まれています。
デフォルトでは下記の通りの設定となっており、api.php には`VerifyCsrfToken`が入っていないため CSRF 保護が機能しておりません。

```php:kernel.php
protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\Session\Middleware\AuthenticateSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],

        'api' => [
            'throttle:api',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
```

## 検証

実際にどのような動きになるのか curl コマンドを使用して検証してみます。

### web.php の場合

事前準備としてエンドポイントを設定します。

- ルーティング

```php:web.php
Route::post('/test', [TestController::class, 'test']);
```

- コントローラーはリクエストをそのまま返却する処理とします

```php:TestController.php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class TestController extends Controller
{
  function test(Request $request)
  {
    return $request;
  }
}
```

```
curl -X POST -H "Content-Type: application/json" -d '{"name":"太郎"}' http://localhost:8000/test
```

```
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">

        <title>Page Expired</title>
```

CSRF 保護の機能が有効になっているため、「Page Expired」が表示されています。

### api.php の場合

上記と同様に事前準備としてエンドポイントを設定します。

- ルーティング

```php:web.php
Route::post('/test', [TestController::class, 'test']);
```

- コントローラー

```php:TestController.php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class TestController extends Controller
{
  function test(Request $request)
  {
    return $request;
  }
}
```

```
curl -X POST -H "Content-Type: application/json" -d '{"name":"太郎"}' http://localhost:8000/api/test
```

```
POST /api/test HTTP/1.1
Accept:         */*
Content-Length: 17
Content-Type:   application/json
Host:           localhost:8000
User-Agent:     curl/7.64.1

{"name":"太郎"}
```

CSRF 保護が有効になっていないため、外部から POST でアクセスできます。

## おわりに

PHP のルーティングファイルについてまとめました。
CSRF 保護のデフォルト設定が異なっており、主な使い分け方としては下記の通りとなります。

- web.php
  画面に表示するようなルーティングを設定する場合に使用する
- api.php
  外部からの HTTP リクエストを受けて値を返却する場合に使用する
