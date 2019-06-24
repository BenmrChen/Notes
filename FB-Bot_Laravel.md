![](https://i.imgur.com/SYJ7xyN.png)
```
前一篇文章已經說明了Node.js的寫法及認知
這篇在以PHP重寫一次後，是針對兩個部份再做進一步闡述
1. 串接FB Messenger Bot的歷程
2. 以PHP重寫時遇到的問題及解法
```
[前一篇文章請點我](https://benmrchen.github.io/FB-Bot/)

<!-- more -->
# [PART 1] 串接FB Messenger Bot歷程
1. FB Messenger BOT 流程/概念
2. FB 範例研究
    - Node.js
    - Linux curl
3. Deploy
    - AWS C9/Heroku: 失敗
    - ngrok: 成功
4. 繼續完成.js，確定可以 echo
5. 回來重整全部流程
![](https://i.imgur.com/3bkiSQj.jpg)
6. 實作 Laravel   
    -  CSRF 檢測->放到api.php
    -  取URI、Body的值
    -  res: 用return response 403
    -  JSON.stringify(webhook_event): 將object轉成字串
    -  POSTMAN 打API
    -  多維陣列
    -  Complex (curly) syntax: https://www.php.net/manual/en/language.types.string.php
    -  Guzzle
        - https://stackoverflow.com/questions/22244738/how-can-i-use-guzzle-to-send-a-post-request-in-json
        - https://guzzle.readthedocs.io/en/latest/request-options.html?highlight=json#body

# [PART 2] 以PHP重寫時遇到的問題及解法
-  CSRF 檢測問題
    - 問題: 由於一開始是直覺地將code寫在web.php裡，沒想到Laravel內default
    會將web.php內的route做CSRF Middleware檢測，以致於會出現類似以下畫面
    ![](https://i.imgur.com/k7IQkJ0.png)
    - 解法
        1. Web 端解法: 可以參考[官方文件](https://laravel.tw/docs/5.0/routing#csrf-protection)
        2. POSTMAN 解法: 可以參考[這裡](https://ithelp.ithome.com.tw/articles/10208593)
        作法是先正常進入網頁的首頁，在取得`token`後，放入POSTMAN裡的header (key:X-XSRF-TOKEN; value:<token>) 
        3. 將code放到`api.php`裡: 這也是筆者的做法
    - PS. CSRF (Cross-site request forgery,跨網站請求偽造)保護是Laravel
    提供的一個簡易方法，用來保護網站不受到CSRF攻擊，作法為:
    Laravel 為所有上線使用者的 Session 產生一個 `CSRF “token”`。該 token 用來驗證使用者為實際發出請求至應用程式的使用者。
-  取Request String、Body的值
    - Request String: 
        - Eg. http://newmessengerl.test/api/webhook?hub.verify_token=newMessengerL&hub.challenge=CHALLENGE_ACCEPTED&hub.mode=subscribe11
        - 取法: `$token = request('hub_verify_token');`直接用request並指定key來取value，
        須特別注意`.`會被解釋成`_`，在鍵入key時要小心。若要確定實際打入api的request string
        是什麼，可以鍵入`dd(request()->all()`來檢查
    - Body:
        - E.g.
        ```
        {
            "object": "page",
            "entry": [
                {
                    "messaging": [
                        {
                            "sender": {
                                "id": "2205210249526222"
                            },
                            "recipient": {
                                "id": "1421661017849456"
                            },
                            "timestamp": 1561096169867,
                            "message": {
                                "mid": "iLrY0OdHj8QnOstgvefOZMiEYWEsnLoC3bTCQVJLbL9YB6i4Fkycy_tLcptOHcflo0G6oadCpstBOBkKvo52PA",
                                "seq": 0,
                                "text": "111"
                            }
                        }
                    ]
                }
            ]
        }
        ```
        - 方法一: 直接使用`request()->input()`來取值，取得的會是`array`
        
        - 方法二: 呼叫符合`request`的變數並使用它來取值，取得的會是`object`
        ```php
        <?php
        Route::post('/webhook', function (Request $request) {
          $a = $request->object; // 這邊的$a會是page (string)
          foreach ($request->entry as $entry) {
              $webhook_event = $entry['messaging'][0];
              $sender_psid = $webhook_event['sender']['id'];   
              // $webhook_event及$sender_psid是用array來取值(多維陣列取法)
          }
        });
        ```

-  JSON.stringify(webhook_event): 將object轉成字串
    - 原因: 想要模擬User在FB BOT鍵入訊息時會傳什麼東西給Server, 藉以用POSTMAN來測試API
    - 解法: 承接上一題 若想要知道在JS value的原JSON string是什麼可以使用
    `JSON.stringify(webhook_event)`，同時可以用`console.log(JSON.stringify(webhook_event));`
    來印出來在terminal裡 確定JSON格式的內容是什麼後再置入POSTMAN內試打API
-  res: 用return response 403
    - `return response('',Response::HTTP_FORBIDDEN);` 
    - `return response('', 403);`
    - 以上兩者結果相同但前者更readable
-  POSTMAN 打API
    - 以以下內容為例(node.js)
    ```
    request({
        "uri": "https://graph.facebook.com/v2.6/me/messages",
        "qs": { "access_token": page_access_token },
        "method": "POST",
        "json": request_body 
    }); 
    ```
    - 須要在POSTMAN 鍵入的內容為:
        - uri: 置入URI欄位
        - method: 點選正確的method
        - qs: `Query String`，置入`Params`裡，key為"access_token", value 為 page_access_token
        - 'json': 置入`body`，格式選`application/json`，request_body是上面code寫的變數如下
            ```
              let request_body = {
                "recipient": {
                  "id": sender_psid
                },
                "message": response
              }
            ```

-  Complex (curly) syntax
    - 另一種在雙引號內表達變數+字串的表示法
    - Eg. 以下兩個`text`的寫法是一樣的
        ```php
        <?php
        $response = [
        'text'=>"Test{$webhook_event['message']['text']}",
        'text'=>"Test" .$webhook_event['message']['text'],
        ];
        ```
    - https://www.php.net/manual/en/language.types.string.php
-  Guzzle
    - Laravel 用來發`Http Request`的套件
    - 表示方式 
        ```php
        <?php
        $client->post($uri, [
                GuzzleHttp\RequestOptions::QUERY=>["access_token"=>"<yourToken>"],
                GuzzleHttp\RequestOptions::JSON => $request_body
         ]);
        ```
    - https://stackoverflow.com/questions/22244738/how-can-i-use-guzzle-to-send-a-post-request-in-json
    - https://guzzle.readthedocs.io/en/latest/request-options.html?highlight=json#body
