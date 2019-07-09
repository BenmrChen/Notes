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
 
 - Migration: 建立一個pivot table，讓兩個多對多的表產生關聯
    - pivot table 的名字要注意，預設是要照英文排序來寫 比如說User和Group
    G排的比較前，所以要寫成group_user (當然也可以指定，然後放在關聯的第二個變數)
 - ref.: 建一個pivot table
     - https://laracasts.com/series/eloquent-relationships/episodes/5
     - 7:00開始
    ```php
    <?php
    
    class CreateGroupUserTable extends Migration
    {
        public function up()
        {
            Schema::create('group_user', function (Blueprint $table) {
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
            Schema::dropIfExists('group_user');
        }
    }
    ```
    
- route寫法:
    ```php
    <?php
    
    use App\authPractice\Entity\Group;
    use App\authPractice\Entity\User;
    use Illuminate\Http\Request;
    
    /*
    |--------------------------------------------------------------------------
    | API Routes
    |--------------------------------------------------------------------------
    |
    | Here is where you can register API routes for your application. These
    | routes are loaded by the RouteServiceProvider within a group which
    | is assigned the "api" middleware group. Enjoy building your API!
    |
    */
    
    Route::middleware('auth:api')->get('/user', function (Request $request) {
        return $request->user();
    });
    
    Route::get('/', function(){
    //    $user = User::where('id', '1')->first();
    //    or
    //    $user = User::find(1); //和上面的一樣 find() 方法是用來取指定id的資料 這邊就是取 id=1 的資料
    
    //    dd($user->groups);
        // 這裡應該要用first()而不是get(); 因為first()會把那個資料那拿出來，為一個model, 可以再接->groups
        // 而get()會拿一個collection，因此沒有->groups這個方法
        // 這邊關聯到groups必須加s是因為在User.php裡面的function name裡有加s
    
    //    $groups = Group::where('id', '1')->first();
    //    $groups = Group::find(2);
    //    dd($groups->users);
    //    確認可以多對多取到值
        return response('success');
    });

    ```
    
- https://laravel.tw/docs/5.2/eloquent-relationships#many-to-many
