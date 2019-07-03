PayMeChien - Model
===    
#### 位置
- 放在 `~/projectName/app/`資料夾下
    * 這邊是放在`~/PayMeChien/app/Shop/Entity

#### 使用 artisan 建立 model
- 指令: `php artisan make:model User`
- 同時產生migration:
    ```
    php artisan make:model User --migration
    or
    php artisan make:model User -m
    ```
- https://laravel.tw/docs/5.2/eloquent#eloquent-model-conventions

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
    - e.g. 在 controller 裡的最上方要寫 use App\Shop\Entity\User 

### 多對多關聯
![](https://i.imgur.com/y3bMQVZ.png)
- `belongsToMany`
- 得先建一個pivot table
    - https://laracasts.com/series/eloquent-relationships/episodes/5
    - 7:00開始
- code: 要把User和Group這兩個model給多對多關聯起來
    ```php
    <?php
    
    namespace App\authPractice\Entity;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {

        // 建立與Group的多對多關聯
        public function groups()
        {
            return $this->belongsToMany('App\authPractice\Entity\Group');
        }
    }
    ```
    
    ```php
    <?php
    namespace App\authPractice\Entity;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Group extends Model
    {
        public function users()
        {
            //建立與User的多對多關聯
            return $this->belongsToMany('App\authPractice\Entity\User');
        }
    }
    ```
 
 - Migration
    ```php
    <?php
    
    class CreateGroupUserTable extends Migration
    {
        public function up()
        {
            Schema::create('user_group', function (Blueprint $table) {
                $table->increments('id');
                $table->unsignedInteger('user_id');
                $table->unsignedInteger('group_id');
                $table->timestamps();
            });
        }
    
        /**
         * Reverse the migrations.
         *
         * @return void
         */
        public function down()
        {
            Schema::dropIfExists('user_group');
        }
    }
    ```
    
- https://laravel.tw/docs/5.2/eloquent-relationships#many-to-many
