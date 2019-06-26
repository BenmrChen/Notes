# Terminal Command
- curl -X GET "localhost:1337/webhook?hub.verify_token=newMessenger&hub.challenge=CHALLENGE_ACCEPTED&hub.mode=subscribe"
    - `-X`: 定義方法
    - `URI`的內容包括
        1. `localhost`: domain
        2. `:1337`: port 號
        3. `/webhook`: path
        4. `?`: 使用`GET`時後面要帶出`Query String`，這個問號就是用來帶`Query String`用的
        5. `hub.verify_token=newMessenger`: 前面`hub.verify_token`是 key, 後面`newMessenger`是 value
        6. `&`: 分隔 Query String 用
        7. `hub.challenge=CHALLENGE_ACCEPTED`: 同前述
        8. `&`: 同前述
        9. `hub.mode=subscribe"`: 同前述

- curl -H "Content-Type: application/json" -X POST "localhost:1337/webhook" -d '{"object": "page", "entry": [{"messaging": [{"message": "TEST_MESSAGE"}]}]}'
    - `-H`: 定義 header
        - 這邊是把 content-type 定義為 json
    - `-X`: 定義方法
    - `-d`: 定義 data
        - 可以透過 postman，點 body，選項選 raw
          ```
          {
          	"object": "page", 
          	"entry": [
          		{
          			"messaging": [
          				{
          					"message": "TEST_MESSAGE"
          					
          				}]
          			
          		}
          		]
          }
          ```  
          
        - 可以看到最外圍是一個`object`，裡面有兩個東西，一個叫"object"，一個叫"entry"，裡面就一直包array和object

# code
   ```js
    'use strict';
    // Imports dependencies and set up http server
    const
      express = require('express'),
      bodyParser = require('body-parser'),
      app = express().use(bodyParser.json()), // creates express http server
      port = 5000,
      // 把原本是process.env.port (應該是要require process檔案 去指向裡面的.env，再指向以port為key的值) 直接改掉寫死 (node.js裡面的點點大概可以想成php的->) 
      page_access_token = 'EAAh84HrF238BAEsGvODMhMVvUbk8jT4czB99LTwM7lp2B6rt8bfaA4leyF2dU7e99VZCvSNS48eRAPzTnve8jN9Xhkz8IPJNX7IIvhJ44sUWiAHYL6CMVBa9QJ8yyKBDOvTak0Q0P4QMv9ssKD8rT72DiI8oPV16UZCJ4HZAwZDZD';
      // 這邊原本是要設環境變數 後來是直接把它寫成常數放這
    // Sets server port and logs message on success
    app.listen(port || 1337, () => console.log('webhook is listening'));
    
    // Creates the endpoint for our webhook 
    app.post('/webhook', (req, res) => {  
                        // 這是個callback，可以想像成前面有function以及function name被省略了，而裡面有兩個參數，一個是req (request)，一個是res (respond)
      let body = req.body;
                        // 把req參數裡的body存到body變數裡
      // Checks this is an event from a page subscription
      if (body.object === 'page') {
           // 可以想成body->object的值要等於'page'
        // Iterates over each entry - there may be multiple if batched
        body.entry.forEach(function(entry) {
           // body->entry，並foreach出來，可以想成foreach (entry as entry) {},後面的花括號才是要做的事
          // Gets the message. entry.messaging is an array, but 
          // will only ever contain one message, so we get index 0
          let webhook_event = entry.messaging[0];
          console.log(webhook_event);
          
          // Get the sender PSID
          let sender_psid = webhook_event.sender.id;
          console.log('Sender PSID: ' + sender_psid);
    
          // Check if the event is a message or postback and
          // pass the event to the appropriate handler function
          if (webhook_event.message) {
            handleMessage(sender_psid, webhook_event.message);        
          } else if (webhook_event.postback) {
            handlePostback(sender_psid, webhook_event.postback);
          }      
        });
    
        // Returns a '200 OK' response to all requests
        res.status(200).send('EVENT_RECEIVED');
      } else {
        // Returns a '404 Not Found' if event is not from a page subscription
        res.sendStatus(404);
      }
    
    });
    
    // Adds support for GET requests to our webhook
    app.get('/webhook', (req, res) => {
      // 也是一樣有callback, 傳req和res進來，只是是用query string傳，傳進來後會變成array
      // Your verify token. Should be a random string.
      let verify_token = "newMessenger"
        
      // Parse the query params
      let mode = req.query['hub.mode'];
      // 由於透過 query string 傳進來會是array，所以這邊用array的方式取值 (key是'hub.mode', value在此例是'subscribe')
      let token = req.query['hub.verify_token'];
      let challenge = req.query['hub.challenge'];
        
      // Checks if a token and mode is in the query string of the request
      if (mode && token) {
        // Checks the mode and token sent is correct
        if (mode === 'subscribe' && token === verify_token) {
          
          // Responds with the challenge token from the request
          console.log('WEBHOOK_VERIFIED');
          res.status(200).send(challenge);
        
        } else {
          // Responds with '403 Forbidden' if verify tokens do not match
          res.sendStatus(403);      
        }
      }
    });
    
   ```
   - PS. 上面的`foreach`在粉專key "Test" 的話會把資料都印出來 
    ```
    {
      sender: { id: '2205210249526222' },
      recipient: { id: '1421661017849456' },
      timestamp: 1560922272916,
      message: {
        mid: '6To75VlOa8MhW6BLs3xGHMiEYWEsnLoC3bTCQVJLbL8yaC7w3TRkv3e6sj86MI7gHyzOJF4uAkrxmmSczKEl-A',
        seq: 0,
        text: 'Test'
      }
    }
    ```

# 參考資料
- Soj's URI: https://hackmd.io/ZKt-vqQtQSCJtPqhcFZwng
- ngrok: https://blog.techbridge.cc/2018/05/24/ngrok/

# Postman
- https://7941d75b.ngrok.io/webhook?hub.verify_token=newMessenger&hub.challenge=CHALLENGE_ACCEPTED&hub.mode=subscribe

# 足跡
- FB Messenger BOT 流程/概念
- FB 範例研究
    - Node.js
    - Linux curl
- Deploy
    - AWS C9/Heroku: 失敗
    - ngrok: 成功
        - 指令: `./ngrok http 5000` (目前沒有把ngrok放到全域指令XD)
- 繼續完成.js，確定可以 echo
- 回來重整全部流程
- 實作 Laravel   
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
# 流程
- 概覽
![](https://i.imgur.com/ealbPsI.png)
User 透過 Messenger 傳訊息到 Facebook Server, Facebook Server 會打一個`Post Request`，把`event`傳給 
`Webhook endpoint` ，在 Business Server 內邏輯判斷是什麼`event`並形成`request body`，
然後將打一個包括`request body`的`Post request`給`Send API`通知 Facebook Server 回覆 User

- Component
    - PSID(Page-scoped ID): 一個 User 連一個 Messenger Bot 時會被 Messenger Platform 指派一個 PSID，
    這可以確保每個 User 和 Bot 的互動是一對一的關係。
    - Send API: `Business Server`在收到`Facebook Server`透過`Webhooks`傳來的`event`後，
    可以透過`Send API`來回傳東西去`Facebook Server`，內容可以是`text`、`image`、`file`等
        - URL: https://graph.facebook.com/v2.6/me/messages
    - Webhook: 一個`end point`，通常是`xxxsite/webhook`，用來接收`Facebook Server`傳來的`event`
    並根據其性質來看它是哪哪種`webhook event`。至少會用到`messages`和`messaging_postbacks`這兩種 webhook events

- Steps
    1. Create FanPage
    2. Create Facebook App
    3. Add Messenger Platform to App
    4. Coding
        - 建立`Http server` (等於我們的Web server?)
        - 加入 webhook verify token
            - 自行設定`verify token`
            - 在接收到`GET Request`之後，驗證`verify token`是否一致，若是，則`subscribe` webhook 到
        - 加入 `webhook endpoint` 以接收`POST Request`傳來的`event`
        
    5. Test
        - 啟動: $node index.js
        - 打`GET Request`以 subscribe webhook: `$ curl -X GET "localhost:1337/webhook?hub.verify_token=<YOUR_VERIFY_TOKEN>&hub.challenge=CHALLENGE_ACCEPTED&hub.mode=subscribe"`
            - 若正確: `WEBHOOK_VERIFIED `、`CHALLENGE_ACCEPTED`
        - 打`POST Request`以測試 webhook: `$ curl -H "Content-Type: application/json" -X POST "localhost:1337/webhook" -d '{"object": "page", "entry": [{"messaging": [{"message": "TEST_MESSAGE"}]}]}'`
            - 若正確: `TEST_MESSAGE `、`EVENT RECEIVED`
    6. Deploy
        - 使用`ngrok`建立HTTPS連線
            - https://dashboard.ngrok.com/
    7. Configure the webhook for app
        - Callback URL: 貼上`ngrok`的URL/webhook
        - Verify Token: 貼上先前自訂的 Token
        - 若以上兩個都正確，在點選`驗證`後，FB會發一個`GET Request`確認並儲存下來，
        此時應已將 app 給 subscribe 至粉專，在粉專 Messenger 已可 key 訊息
    8. 建立自動回覆 Bot 相關 coding
        - 將環境變數設為連結粉專時取得的`Auth`，這將在打 POST Reqest 給 
        Send API時使用
        - 建立 `handleMessage`、`callSendAPI`兩個 function 來建立
        判別`event type` 及 call `Send API`
        - 取得 `PSID`
        - 在`handleMessage`內區別 `event type`是`messages`還是`messaging_postback`，
        若是`messages`則建立`response`
        - 透過 `callSendAPI` 來建立`request body`並利用`POST request`打到
        `https://graph.facebook.com/v2.6/me/messages`來 trigger `Send API`
        這邊就會須要使用粉專的`Auth Token`
    9. 完成，此時FB 粉專 Bot 應可以正常 echo 訊息給 User。

# PS.
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
