PayMeChien - Notes
===
## Controller
#### 存放位置
* `~/projectName/app/Http/Controllers` 資料夾內
#### 生成
* 使用 artisan
* e.g.
    ```
    php artisan make:controller UserAuthController 
    ```
    
    結果: 產生 **app/Http/Controllers/UserAuthController.php** 控制器檔案

    
#### namespace
* 用途: 標示本檔案放在什麼資料夾
* 寫法: 用右斜線分開 
* e.g.:
    
    ```php
    <?php
    namespace App\Http\Controllers;
    ``` 
    
* ps. 注意大小寫
    * 在PhpStorm裡會自動區分大小寫，但EC2不會

#### 方法 (function)
##### 寫法: public function(){}
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
#### 常用工具
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
        }
        ```
        - `firstOrFail()`: 在使用方法`where`限制條件之後，用`firstOrFail`這個方法去限制只取第一筆資料，
        當沒有找到資料時，則會丟出例外錯誤訊息
    5. 使用`Eloquent`存資料
        ```
        User::create($input);
        ```
        使用已經建好的 User Eloquent 來存資料 (參考下面 Model - Eloquent)
    
    

## Migration
#### 存放位置
- `database/migrations`資料夾內
#### 使用 Artisan 建立 migration 
  ```
  $ php artisan make:migration create_users_table
  $ php artisan make:migration create_transactions_table
  $ php artisan make:migration create_services_table
  ```
  根據 artisan 建立好的 migration 檔案會在 `database/migrations` 裡面，裡面要注意的是:
  * `public function up()`: 執行 migrate 後的動作
  * `public function down()`: 執行 rollback 後的動作
  * 可以指定要建什麼 table、欄位名、屬性是什麼、長度
  * example:
      ```php
      <?php
      class CreateUsersTable extends Migration
      {
          /**
           * Run the migrations.
           *
           * @return void
           */
          public function up()
          {
              Schema::create('users', function (Blueprint $table) {
                  // USER編號
                  $table->increments('id');
                  // Email
                  $table->string('email', 150);
                  // 密碼
                  $table->string('password', 60);
                  // 暱稱
                  $table->string('nickname', 50);
                  // 帳號類型
                  $table->string('type',1)->default('G');
                  // 建立/更新時間
                  $table->timestamps();
                  // 唯一鍵值索引
                  $table->unique(['email'],'user_email_uk');
              });
          }
      
          /**
           * Reverse the migrations.
           *
           * @return void
           */
          public function down()
          {
              Schema::dropIfExists('users');
          }
      }
      ``` 

#### 常用指令
* 執行 migration 檔案: `php artisan migrte`
* 恢復上一版本的 migration: `php artisan migrate:rollback`
* 清除所有 migration 版本: `php artisan migrate:reset`
* 重設且重設所有 migration 檔案: `php artisan migrate:refresh` (fresh也可)
    

## Model - Eloquent
#### 位置
- 放在 `~/projectName/app/`資料夾下
    * 這邊是放在`~/PayMeChien/app/Shop/Entity

#### 使用 artisan 建立 model
- 指令: `php artisan make:model User`

#### 一個 table 對應到一個 Eloquent
- 內容 example
    ```php
    <?php
    namespace App\Shop\Entity;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model {
        // 資料表名稱
        protected $table = 'users';
    
        // 主鍵名稱
        Protected $primaryKey= 'id';
    
        // 可大量指定異動column
        protected $fillable = [
            "email",
            "password",
            "type",
            "nickname",
        ];
    }
    ```
    命名空間`namespace`要用反斜線 標示出目前檔案的資料夾位置，注意大小寫和 controller 的要一致 
    - e.g. 在 controller 裡的最上方要寫 use app\Shop\Entity\User 



## View
#### @extends: 繼承指定模版
* e.g. `extends('layout.master`)

#### @section: 傳值
* 兩種用法
    1. @section(要傳的片段, 要傳的內容)
        - e.g. `@section('title',$title)`
    2. @section, @endsection: 指定要傳的片段，接著再把要傳的內容包在其中
        * e.g. 
            ```php
            <?php
            @section('content')
             // What you want to express
             abc
            @endsection
            ```

#### 直接引用其他模版
* @include('components.validationErrorMessage')
    
#### CSRF
* 在 blade 裡須要塞一段 `{!! csrf_field() !!}` (Laravel 的保護機制)
       
       

    