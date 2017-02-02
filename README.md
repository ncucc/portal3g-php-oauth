# portal3g-php-oauth

這個是用 Laravel 5.2 的系統連上中央大學第三代 portal 系統 (Portal 3G) 使用的 OAuth 設定, 使用方式如下:

修改 composer.json, 加上

```json
    "require": {
        "linuzilla/portal3g-php-oauth": "dev-master",
        "oriceon/oauth-5-laravel": "dev-master#3f6c663b48c9878cc1d8c56e16ad6c2c3d019dc8"
    },
```
執行 composer update 將需要的 package 安裝進來
修改 config/app.php 的設定 (請參考 "oriceon/oauth-5-laravel" 的說明
```php
'providers' => [
    // ...
    Artdarek\OAuth\OAuthServiceProvider::class,
]

'aliases' => [
    // ...

    'OAuth' => Artdarek\OAuth\Facade\OAuth::class,
]
```
然後執行 php artisan vendor:publish, 這個動作會產生 config/oauth-5-laravel.php 檔案, 編輯這個檔案加入我們設定
```php
'NCUPortal' => [
    'client_id'     => '',
    'client_secret' => '',
    'scope'         => [ 'xxx', 'xxx' ]
 ],
 ```
 當然, client_id 及 client_secret 是自 portal3g 取得的, scope 則是系統希望使用者授權的.
 
 基本上, 這樣就算完成, 剩下的部份是應用程式自己要去配合需要去撰寫的
