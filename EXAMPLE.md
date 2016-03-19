這份文件是提供完全不知道怎麼開始用的一點說明, 文件假設從空的專案開始, 而且專案本身沒有提供 Portal 3G 以外的認證方式.

我們可以寫一個 SigninController.php, 放在 app/Http/Controllers 目錄下, 這隻登入程式大概像
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

use App\Http\Requests;
use Auth;
use App\User;

class SigninController extends Controller
{
    const CALLBACKURL = "https://xxxxxxx/signin/callback";
        
	  public function signin(Request $request) {
		    $oauth2 = \OAuth::consumer('NCUPortal', self::CALLBACKURL);

		    $url = $oauth2->getAuthorizationUri();
		    return redirect((string)$url);
	  }

	  public function callback(Request $request) {
        $code = $request->get('code');

        if ($code == null) {
            return view('signin.failed')->with("message", "cancel");
        } else {
            $portalsvc = \OAuth::consumer('NCUPortal', self::CALLBACKURL);
        
            try {
                $token = $portalsvc->requestAccessToken($code);
                                
                $result = json_decode($portalsvc->request('/apis/oauth/v1/info'), true);
                
                if (array_key_exists ('identifier', $result)) {
                    $user =  User::firstOrCreate([
                            'portal_id' => $result['id'],
                            'identifier' => $result['identifier']
                    ]);
                                        
                    Auth::login($user);
                    return redirect()->intended('/');
                } else {
                    return view('signin.failed')->with("message", "Need to auhorize your identifier");
                }
            } catch (\Exception $e) {
                return view('signin.failed')->with("message", "Exception");
            }
        }
    }
        
    public function logout() {
        Auth::logout();

        return redirect(property_exists($this, 'redirectAfterLogout') ? $this->redirectAfterLogout : '/');
    }
}
```
登入失敗的 View 請自行發揮, 接下來要註冊在 app/Http/routes.php
```php
Route::group(['middleware' => ['web']], function () {
    Route::get('/signin', 'SigninController@signin');
    Route::get('/signin/callback', 'SigninController@callback');
    Route::get('/logout', 'SigninController@logout');
});
```
修改 database/migration 裡的 create_users_table, 這個修改只是示意而已
```php
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->increments('id');
            $table->string('portal_id')->unique();
            $table->string('identifier');
            //$table->string('name');
            //$table->string('email')->unique();
            // $table->string('password');
            $table->rememberToken();
            $table->timestamps();
        });
    }
```
另外還要修改 app/User.php, 如
```php
    protected $fillable = [
        // 'name', 'email', 'password',
        'portal_id', 'identifier'
    ];
```
然後做一次 php artisan migration

大概就這樣吧 ...
