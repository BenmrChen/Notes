# 定義:
- https://developer.mozilla.org/zh-TW/docs/Web/HTTP/CORS#%E8%AA%B0%E6%87%89%E8%A9%B2%E9%96%B1%E8%AE%80%E9%80%99%E7%AF%87%E6%96%87%E7%AB%A0%EF%BC%9F

# 條件
- 例如來自於不同網域（domain）、通訊協定（protocol）或通訊埠（port）的資源時，會建立一個跨來源 HTTP 請求（cross-origin HTTP request）。

# 解法 
- 在header加入 `Access-Control-Allow-Origin = * ` 加好加滿
- Eg.
    ```php
    <?php
    Route::post('/test', function() {
        $name = request()->input('name');
        return response("Hi $name")
    //        ->header('Access-Control-Allow-Methods','*')
            ->header('Access-Control-Allow-Origin', '*');
    //        ->header('Access-Control-Expose-Headers','*');
    });
    ```

其他文章
- https://developer.mozilla.org/zh-TW/docs/Web/HTTP/Server-Side_Access_Control
- Laravel HTTP 請求/response
    - https://laravel.tw/docs/5.2/requests
    - https://laravel.com/docs/5.8/responses
- https://kjj6198.github.io/2019/01/18/cors-and-cookie/
