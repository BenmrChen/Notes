PayMeChien - View
===
## @extends: 繼承指定模版
* e.g. `extends('layout.master`)

## @section: 傳值
* 兩種用法
    1. @section(要傳的片段, 要傳的內容)
        - e.g. `@section('title',$title)`
    2. @section, @endsection: 指定要傳的片段，接著再把要傳的內容包在其中
        * e.g. 
            ```php
            <?php
            @section('content')
             // What you want to express
             abc
            @endsection
            ```

## 直接引用其他模版
* `@include('components.validationErrorMessage')`
    
## validationErrorMessage.blade.php
```php
<?php
@if($errors)
  <ul>
    @foreach($errors->all() as $err)
      <li> {{ $err }}</li>
    @endforeach
  </ul>
@endif
```

前面在 controller 裡面是寫
```
 $error_message = [
                'msg' =>
                    '帳號或密碼錯誤',
            ];
 return redirect('/user/auth/sign-in')
        ->withErrors($error_message)
```

`$error_message`會被`redirect`的`withErrors`方法帶到指定`blade`去，而`withErrors`方法會設一個`default`的key去對應
`$error_message`這個 value，並包成一個名為`$errors`的object(dd出來會是一個`bag`) 接著在`blade`裡判定若`$errors`不為空，
就可以用`->all()`的方法去取值並`foreach`出來印出



## CSRF
* 在 blade 裡須要塞一段 `{!! csrf_field() !!}` (Laravel 的保護機制)
* Soj iT邦幫忙:
https://ithelp.ithome.com.tw/articles/10208593

       
       

    