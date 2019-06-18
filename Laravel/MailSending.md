# 配置
- 安裝 Guzzle HTTP 函式庫: `~/projectName/ $ composer require guzzlehttp/guzzle`
- .env 設定
    ```
    MAIL_DRIVER=smtp
    MAIL_HOST=smtp.gmail.com
    MAIL_PORT=587
    MAIL_USERNAME=SojLaravel99@gmail.com
    MAIL_PASSWORD=Soj778899
    MAIL_ENCRYPTION=tls
    ```
    Gmail 要在帳戶內做降低安全層級 (但似乎沒做、也沒用兩階段驗證也可以)
    
# 寄送驗證信
### controller 內
- ```php
    <?php
     // 處理註冊資料
        public function signUpProcess()
        {
            // 接收輸入資料
            $input = request()->all();
         
            // 寄送註冊通知信
            $mail_binding = [
            'nickname' => $input['nickname'],
            'email' => $input['email'],
            ];
          
            Mail::send('email.signUpEmailNotification', $mail_binding, function($mail) use ($input){
                      // 收件人
                      $mail->to($input['email']);
                      // 寄件人
                      $mail->from('PayMeChien@gmail.com');
                      // 郵件主旨
                      $mail->subject('恭喜您，已成功註冊「PayMeChien」');
                    });
          $message = [
              'msg' => '恭喜您已成功註冊! 您將會收到一封確認郵件，謝謝。'];
              //重新導向到登入頁
               return redirect('/user/auth/sign-in')
                      ->withErrors($message);
        }
  ```
  邏輯: 取得輸入資料->把需要的資料丟到`$binding`裡傳到 email 的 blade 
  (放在`email`資料夾內，名為`signUpEmailNotification`)，接著再導回 `sign-in` page，並帶上`$message`的成功訊息
  - PS. 若之後用 `queue/job` 來派送的話，`Mail::send`那段要改成`SendSignUpMailJob::dispatch($mail_binding);`
    指的是要指派一個job給SendSignUpMailJob，並傳`$mail_binding`這個變數過去