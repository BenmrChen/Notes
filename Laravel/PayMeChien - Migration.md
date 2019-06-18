PayMeChien - Migration
===
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

