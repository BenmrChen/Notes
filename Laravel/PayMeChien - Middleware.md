- 建立 middleware
    - `php artisan make:middleware AuthUserAdminMiddleware`

- 寫middleware
    - 概念
        - 位置: App\Http\Middleware
        - 先預設不能存取，接著從 session 取得會員編號後去db撈資料 確認是否為管理員身份
        若是 則可存取 否則反之
    - code
        ```php
        <?php
        namespace App\Http\Middleware;
        
        use App\Shop\Entity\User;
        use Closure;
        
        class AuthUserAdminMiddleware
        {

            // 處理請求
            public function handle($request, Closure $next)
            {
                // 預設不允許存取
                $is_allow_access = false;
        
                // 取得會員編號
                $user_id = session()->get('user_id');
        
                if (!is_null($user_id)) {
                    // session 有會員編號，可取得會員資料
                    $User = User::findOrFail($user_id);
        
                    if ($User->type == 'A') {
                        // 確認是管理者，可存取
                        $is_allow_access = true;
                    }
                }
        
                if (!$is_allow_access) {
                    // 若不允許存取，重新導向至首頁
                    return redirect()->to('/');
                }
        
                // 允許存取，繼續處理下一個請求
                return $next($request);
            }
        }

        ```
 - 註冊 middleware
    - code
        ```
            protected $routeMiddleware = [
                'auth' => \App\Http\Middleware\Authenticate::class,
                //........略.........
                'user.auth.admin' => \App\Http\Middleware\AuthUserAdminMiddlewares::class,
            ];
        ```
        `protected $routeMiddleware`用來放個別的middleware, 可以分別
        指定給不同的route
 
 - 在route中使用middleware
    - 本例中，會須要使用管理者權限的有4個
    - code
    ```
        // 新增商品服務
        Route::get('/create', "ServiceController@serviceCreateProcess")
        ->middleware(['user.auth.admin']);
        
        // 服務管理清單檢視
        Route::get('/manage', "ServiceController@serviceManageListPage")
        ->middleware(['user.auth.admin']);
        // 服務單品編輯頁面檢視
        Route::get('/edit', "ServiceController@serviceItemEditPage")
        ->middleware(['user.auth.admin']);

        // 服務單品資料修改
        Route::put('/', "ServiceController@serviceItemUpdateProcess")
        ->middleware(['user.auth.admin']);
    ``` 