PayMeChien - Notes
===
## Controller
#### 生成
- 使用Artisan
- e.g.
    ```
    php artisan make:controller UserAuthController 
    ```
    結果: 產生 **app/Http/Controllers/UserAuthControllr.php** 控制器檔案

    
#### namespace
- **用途**: 標示本檔案放在什麼資料夾
- *寫法*: 用右斜線分開
- e.g.:
    ```php
  <?php
    namespace App\Http\Controllers;
    ``` 
- ps. 注意大小寫
    - 在PhpStorm裡會自動區分大小寫，但EC2不會

#### 方法 (function)
##### 寫法: public function(){}
    - e.g.:
    ```php
    <?php
    class UserAuthController extends controller {
      public function signUpPage(){
        $binding = [
          'title' => '註冊'
        ];
        return view('auth.signUp', $binding);
      }
    }
    ```
   - p.s.: array的寫法: `"中框號" + "key=>value"` 
   - p.s.: 若要回傳什麼的話就用 `return` 
   - P.s.: `view('auth.signUp', $binding)`的意思是要去找`resourses/views/auth`資料夾裡的`signUp.blade.php`檔 
##### 常用工具
- 接受輸入資料
 `$input = request()->all;`
- 驗證
    1. 建立驗證規則
        ```php
        <?php
        $rules = [
          'nickname' => [
              'required',
              'max:50',
          ],
          'email' => [
              'required',
              'email',
              'max:50'   
          ],
          'password' => [
              'required',
              'same:password_confirmation'  
          ],
          'type' => [
              'required',
              'in:G,A',  
          ],
        ];
        ```    
        - PS. 各個rules 內在指定時要用`''`給包起來
        - PSS. 和指定項相同: `same:xxx`; `'in:G,A'`必須是G或A
        - More: https://laravel.tw/docs/5.2/validation#available-validation-rules
    
    2. 驗證資料
        `$validator = Validator::make($input, $rules);`
        - 使用靜態方法呼叫Validator (前題: 需在上面 use Validator;)
        - 存成物件存在$Validator