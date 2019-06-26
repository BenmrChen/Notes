PayMeChien - Controller
===
# 存放位置
* `~/projectName/app/Http/Controllers` 資料夾內
# 生成
* 使用 artisan
* e.g.
    ```
    php artisan make:controller UserAuthController 
    ```
    
    結果: 產生 **app/Http/Controllers/UserAuthController.php** 控制器檔案

    
# namespace
* 用途: 標示本檔案放在什麼資料夾
* 寫法: 用右斜線分開 
* e.g.:
    
    ```php
    <?php
    namespace App\Http\Controllers;
    ``` 
    
* ps. 注意大小寫
    * 在PhpStorm裡會自動區分大小寫，但EC2不會

# 方法 (function)
## 寫法: public function(){}
   * e.g.:
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
   * PS. array的寫法: `"中框號" + "key=>value"` 
   * PSS. 若要回傳什麼的話就用 `return` 
   * PSSS. `view('auth.signUp', $binding)`的意思是要去找`resourses/views/auth`資料夾裡的`signUp.blade.php`檔 
## 常用工具
* 接受輸入資料
 `$input = request()->all;`
* 驗證
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
        ````
        
        * PS. 各個rules 內在指定時要用`''`給包起來
        * PSS. 和指定項相同: `same:xxx`; `'in:G,A'`必須是G或A
        * More: https://laravel.tw/docs/5.2/validation#available-validation-rules
    
    2. 驗證資料
        `$validator = Validator::make($input, $rules);`
        * 使用靜態方法呼叫Validator (前題: 需在上面 use Validator;)
        * 存成物件存在$Validator
            ```php
            <?php
            if($validator->fails()){
               return redirect('user/auth/sign-up')
                      ->withErrors($validator)
                      ->withInput();
             }
            ```
        * 取出驗證失敗的方法: `If($validator->fails())`
        * 把失敗資料帶著傳回去return redirect的目的地址
           * `withErrors($validator)`
           * `withInput()`
        * 在 blade 顯示 error message
           * 在 route 裡認到'user/auth/sign-up'後就會轉到 signUp.blade，並帶著 $errors 過去
           
           ```php
             <?php
             @if($errors)
                <ul>
                    @foreach($errors->all() as $err)
                    <li>{{ $err }}</li>
                    @endforeach
                </ul>
           ```
           
           * 用 `all` 的方法來把 `$errors` 裡面的資料取出來
           * 取出來之後才能再用`foreach`來把一筆一筆印出
           * 以上語法可以存在`components`資料夾內，並命名為validationErrorMessage.blade
           * 在主 blade 內就可以用 `@include('components.validationErrorMessage')` 把它引起來用
        * 在 blade 帶入 User 上次的 input
            1. 使用 old
            2. example
                ```
                <label for="">
                        暱稱: <input type="text" name="nickname" placeholder="暱稱" value="{{ old('nickname') }}">
                </label>
                ``` 
                
        * 驗證是否唯一
            1. 關鍵字: `unique`
            2. 作法: 
               - 
                 ```
                    // Email
                              'email'=>[
                                  'required',
                                  'max:150',
                                  'email',
                                  'unique:users,email'
                                  // 驗證在資料庫裡，users這個table的email欄位
                              ],
                  ```
                - reference: https://laravel.tw/docs/5.2/validation#validation-quickstart
    3. 密碼加密: 使用Hash
        ```php
        <?php
        $input = request()->all();
        $input['password'] = Hash::make($input['password']);
        ```
        使用靜態方法make把密碼用Hash存到$input裡; 加密時，會使用.env裡的`APP_KEY`當作密鑰進行加密
    
    4. 檢查密碼是否正確
        ```php
        <?php
        public function signInProcess(){
           $input = request()->all();
           $User = User::where('email', $input['email'])->firstOrFail();
           $is_password_correct = Hash::check($input['password'], $User->password);
           if(!$is_password_correct){
               $error_message = [
                   'msg' => ['密碼驗證錯誤']
               ];     
           }
        }
        ```
        
        - `firstOrFail()`: 在使用方法`where`限制條件之後，用`firstOrFail`這個方法去限制只取第一筆資料，
        當沒有找到資料時，則會丟出例外錯誤訊息
        - 也可以使用多個`where`條件來篩資料，如:
        ```php
        <?php
        $User = User:where('email', $input['email'])
                ->where('type','A')
                ->firstOrFail();
        ```
        
        - 密碼檢查`Hash::check($input['password'], $User->password)`回傳為布林值
    5. 使用`Eloquent`存資料
        ```
        User::create($input);
        ```
        使用已經建好的 User Eloquent 來存資料 (參考下面 Model - Eloquent)
## 存到`session`
- code 
    ```php
    <?php
    public function signInProcess(){
      $input = request()->All();
      $rules = ['...'];
      $validator = Validator::make($input, $rules);
      $User = User::where('email', $input['email'])->first();
      // session 紀錄會員編號
      session()->put('user_id', $User->id);
    }
    ```
 
 - 說明   
    - 使用`put`把值放進`session`: `session()->put('key', value)`
 
 ## 重新導向
 - code
    ```php
    <?php
    return redirect()->intended('/');
    ```
 
 - 說明: 重新導向到使用者原來的網頁，若無，則回到首頁`'/'`
 
 ## `session`設定
 - 位置: `config/session.php`
 - 
    
## Typing Hint
- PHP的語法 = 類型約束
- e.g. (這個在`serviceController.php`裡)
    ```php
    <?php
    try {
      DB::beginTransaction();  
      // 建立交易資料
      Transaction::create($transaction_data);
      // 交易結束
      DB::commit();
    } catch (Exception $exception) {
      // 恢復原先交易狀態
      DB::RollBack();
    
      // 回傳錯誤訊息
      $error_message = [
          'msg' => [$exception->getMessage()]
      ];

      return redirect()
              ->back()
              ->withErrors($error_message)
              ->withinput();
    };
    ```

    - `DB::beginTransaction()`:資料庫開始動作，直到DB:commit之前，資料庫都不會變更(也不會被其他的請求變更)，
    意即只有此次的交易可以做些資料異動。
    > 這邊要提到一個蠻重要的 `Transaction`，它的功用就是當我們在執行資料庫內容變更時，控制我們的資料變動是否要真的寫入到DB的一個功能，通常在開發時，
    如果有多步DB內容更動的操作，會建議加上，這樣萬一出錯時，才不會有部分指令寫入，另一部分卻沒有執行到的狀況。
    - `catch (Exception $exception) {}`:
        - 這個`Exception $exception`就是`typing hint`, 意即把`$exception`這個變數指定成`Exception`
        的`class` (class是還沒實例化的，實例化後才會變object)
        - 可以按`command + b`看`Exception`是怎麼被宣告的
        - 下面回傳錯誤訊息的部份，因為$exception這個object有很多東西，我只要error message，
        所以要用`getMessage()`這個方法 (如果沒用的話就會有一堆訊息，包括處理了哪些檔案、第幾行怎麼了blablabla)
            ```
            // 回傳錯誤訊息
            $error_message = [
                'msg' => [$exception->getMessage()]
            ```
    - `try{} + catch{}`
        - 在`try {}`一旦出現`exception`，就會結束執行並跳到`catch {}`
- Reference: 
    - https://www.php.net/manual/en/language.oop5.typehinting.php
    - https://laravel.tw/docs/4.2/database#database-transactions