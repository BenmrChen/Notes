# CSRF
- 定義
    - `SOJ`:
        - 跨站請求偽造（英語：Cross-site request forgery），也被稱為 one-click attack 或者 session riding，通常縮寫為 CSRF 或者 XSRF，是一種挾制用戶在當前已登錄的Web應用程式上執行非本意的操作的攻擊方法。[1] 跟跨網站指令碼（XSS）相比，XSS 利用的是用戶對指定網站的信任，CSRF 利用的是網站對用戶網頁瀏覽器的信任。
        - 攻擊者並不能通過CSRF攻擊來直接獲取用戶的帳戶控制權，也不能直接竊取用戶的任何資訊。他們能做到的，是欺騙用戶瀏覽器，讓其以用戶的名義執行操作。
        - 假設阿美在自己加登入銀行執行轉賬動作，如果伺服器端因為已經確認過阿美的帳號密碼，就同意這次操作。同一時間在另外一邊，有一個壞蛋阿寶，趁著阿美還在登入狀態，也模仿一個銀行轉賬的網路請求，那麼伺服器要怎麼判定這次操作是不是阿美執行的？答案就是透過 session！
    - `我的的理解`: 若有心人士A在一般User B登入之後，拿到了B的header資料，即可仿照header去發request給server，
    進而偽造成B去動作。

- 取得CSRF-token
    - 設環境變數
    - 在`tests`寫code，以自動取得 token
        ```
        pm.environment.set(
            "XSRF-TOKEN",
            decodeURIComponent(pm.cookies.get("XSRF-TOKEN"))
        )
        ```
    - header加入 X-XSRF-TOKEN:{{XSRF-TOKEN}}

- Temporary headers:
    - Postman 自動幫打的 header

- 實作
    - 使用Postman打兩個登入，有兩個session後，若執行buy，系統會認最後一次登入的session去buy
    - header不用另外再加`laravel_session` 因為在登入時就已經存在server了

- `valet secure <專案名>`: 加上SSL
    - SSL 是一個第三方發行的認證，這邊用這個指令只是讓該專案看起來是SSL，但實際上沒有，所以用Postman去打https
    的時候就會無法成功，需要在preference裡把ssl給點掉
- Reference: https://ithelp.ithome.com.tw/articles/10208593?sc=iThelpR
