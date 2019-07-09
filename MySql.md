- zsh 設定 
    - https://ithelp.ithome.com.tw/articles/10207691?sc=iThelpR
- 指令放全域: command not found:laravel
    - https://ithelp.ithome.com.tw/articles/10207960
        - `vim ~/.zshrc`
        - 在最後一句加上 `export PATH=~/.composer/vendor/bin:$PATH`
        - `source ~/.zshrc` : 讀取/更新設定檔

# Route 問題
- GET method 不能取 body，只能取query string
- 若打了valet secure <project name>，用 Postman 打 POST 時會自動轉址，然後出現`301 error`(redirect 錯誤)
看起來是因為valet會轉址時先打一個GET，然後就會跳錯誤

# 有顏色的註解
- 例
    ```
    class AuthUserAdminMiddleware
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        
        public function handle($request, Closure $next)
        {
            return $next($request);
        }
    }
    
    ```
- 以上被 `/** */`包起來的是註解 但是會被其他文件解析 如API Doc.


# Howard 挑戰
一到十個人
可以使用功能的不一樣
要

User 
Group 
Function

可以指派function 給 user 或group
User 可以加群組，可以執行function，要可以判斷可否使用

用or的概念 若是a user 或 a group 就可以執行該function

# mysql
- 查看mysql的安装位置： `$ which mysql`
- 查資訊: `$ brew info mysql`
