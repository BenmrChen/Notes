PayMeChien - Model
===    
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
    - e.g. 在 controller 裡的最上方要寫 use App\Shop\Entity\User 

