# 名詞解釋
- Class 裡的函式 (= function): `Method` aka 方法
- Class 裡的變數 (= variable/parameter): `Property` aka 變數
- Function 裡的變數: `Argument`
> 與人溝通時很重要，名詞定義清楚才好溝通，另外 error message 也是用以上名詞

# 後端傳 result 到前端
- 傳 array，在 Laravel 會自動幫轉 Json
    - 例
        ```
        $test = ["result" => "success",
                  "product: => {"id": 1,
                                "description": "test",
                                     ...
                                }
                ]
        
        // 以上會是一個 array，要傳回前端的時候如下
        
        return response ($test);
        // 在 Laravel 就會自動把該 array 轉為 Json，傳到前端
        ```
        
- Json 為字串
    - 例
        ```
        { "text": "abc",
          "int" : 1
        }
        ``` 
        以上其實只是個符合Json格式的字串(string)
        對我們來說 Laravel 會幫 encode 並傳回前端，而前端會再decode成他們要的東西來使用 
        