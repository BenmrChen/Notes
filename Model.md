
# 監聽查詢事件
- import use Illuminate\Support\Facades\Log;
- 透過 Log::info() 把記錄存在 Log file(路徑：/storage/logs)
     ```php
      <?php
      
      namespace App\Providers;
      
      use Illuminate\Support\Facades\DB;
      use Illuminate\Support\Facades\Log;
      use Illuminate\Support\ServiceProvider;
      
      class AppServiceProvider extends ServiceProvider
      {
          /**
           * 啟動任何應用程式服務。
           *
           * @return void
           */
          public function boot()
          {
              DB::listen(function ($query) {
                  Log::info($query->sql);
                  // $query->bindings
                  // $query->time
              });
          }
      }
     ```

# Query builder vs. Collection
- Code
    ```php
      <?php  
      class TransactionController extends Controller {
        
            public function transactionListPage() {  
                 // 每頁資料量
                 $row_per_page = 10;
                 // 撈取商品分頁資料
                 $TransactionPaginate = Transaction::where('user_id', 15)
                     ->OrderBy('created_at', 'desc')
                     ->with('Service') //順便帶Service資料出來
                     ->paginate($row_per_page);
        //           ->all(); 這邊不能ALL出來是因為下面會傳到bunding後再傳到blade, 接著在blade那邊再用paginate的方法"links()"來取值出來
        //        所以 沒get出來就是query builder, 有的話就是model instance; 前者是class 後者是array (array是key和value組成，而這邊它的value是個class, 所以下面foreach ($TransactionPaginate as $Transaction)才有辦法用$Transaction->total_price取到屬性
        //        6/27補充:沒get出來是query builder，可以繼續使用query builder的鏈結語法(如where, orderBy...)，
        //        但，若get出來了就會是個符合「Illuminate\Database\Eloquent\Collection」的instance，所以它可以使用collection的鏈結語法
      
        // 沒有->all()或get();  會是一個 builder，後面可以再接query builder的鏈結語法
        // 有->all()或get();    會是一個 collection
        // 有->paginate就要用->all(), 若無，則要用->get()   
    ```
- 先前知識: Laravel 資料庫訪查的三種方式: 1.*原生 SQL 查詢* 2.*查詢建構器 Query Builder* 3.*Eloquent ORM* 
    1. 原生 SQL 查詢
        ```php
        <?php
        // 連線到資料庫
        DB::connection('mysql');
        
        //select
        $users = DB::select('select * from users where active = ?', [1]);
        //insert
        DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);
        //update
        $affected = DB::update('update users set votes = 100 where name = ?', ['John']);
        //delete，後面一樣可以接 where 語句設定條件
        $deleted = DB::delete('delete from users where id = ?', [1]);
        //一般陳述式
        DB::statement('drop table users');
        ```
        
        > select 查詢出來的結果，都是以陣列方式存放，而裡面每筆資料都是一個 PHP stdClass 的物件。
        > 1. 由於取出來的資料是陣列，所以如果從資料庫撈出來的資料想再作處理，就要比照陣列的處理模式，例如 foreach 、 while ...
        > 2. 由於每筆資料都是 stdClass，所以可以把存取的欄位當作物件的一個屬性來存取每個欄位的值
    
    2. 查詢建構器
        ```php
        <?php
        // 連線到資料庫，使用原生 sql 語法才需要
        DB::connection('mysql');
        //取得資料
            //原生 sql 語法
            $users = DB::select('select * from users');
            //使用查詢建構器
            $users = DB::table('users')->get();
        ```
        
        > get 方法取得的結果，都是 Illuminate\Support\Collection (集合)實例，每個都是 PHP StdClass 物件。
        > 1. 由於取出來的資料是 集合，所以如果資料想再作處理，可以直接在用集合的鏈結語法處理資料，例如 pluck chunk ...
        > 2. 由於每筆資料都是 stdClass，所以可以把存取的欄位當作物件的一個屬性來存取每個欄位的值
        > ps. stdClass可以看作是一個單獨的class 沒有繼承其他的class
    
    3. Eloquent ORM
        ```php
        <?php
          $TransactionPaginate = Transaction::where('user_id', $user_id)
              ->OrderBy('created_at', 'desc')
              ->with('Service') //順便帶Service資料出來
              ->paginate($row_per_page);
        ```
        
        >Eloquent 回傳的複數資料結果，都會是 Illuminate\Database\Eloquent\Collection 的實例
        這跟「查詢建構器 Query Builder」的方法 get() 一樣。
        >> 不過如果如果沒有用get()或all()之類的方法取出的話，就會*query builder*
        >集合物件 (collection)繼承 Laravel collections，所以它自然也繼承了幾十種用於與底層 Eloquent 模型陣列的優雅方法。
         集合相比陣列強大許多，除了提供 map、sort 或 reduce 等各種遍歷集合的方法外，最大的好處就是可以透過「鏈結語法」的直觀操作。
         
    - Reference:
        - https://ithelp.ithome.com.tw/articles/10209206
        - https://ithelp.ithome.com.tw/articles/10209340
        - https://ithelp.ithome.com.tw/articles/10209583 