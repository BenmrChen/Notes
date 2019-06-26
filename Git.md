- git 多點合成一點
    - master git init
    - master touch .gitignore
    - master git add .gitignore
    - master git commit -m 'init'
    - master git checkout -b Feature
    - Feature touch 1 2 3 4
    - Feature git add 1
    - Feature git commit -m '1'
    - Feature {repeat step 7. 8. for file 2, 3, 4 }
    - Feature git branch Dev
    - Feature git rebase -i --root --onto master Dev
    - {in vim editor} {reword 1, fixup 2, 3, 4}
    - {in vim editor} {rename commint '4 in 1'}
    - Dev git checkout master
    - master git merge Dev
    - master git chechout Feature
    - Feature touch 5 6 7 8
    - Feature git add 5
    - Feature git commit -m '5'
    - Feature {repeat step 17. 18. for file 6, 7, 8 }
    - Feature git branch ToBeMerge
    - Feature git rebase -i {commit 4} --onto Dev ToBeMerge
    - {in vim editor} {reword 5, fixup 6, 7, 8}
    - {in vim editor} {rename commint '8 in 5'}
    - ToBeMerge git checkout Dev
    - Dev git merge ToBeMerge
    
- git .gitignore 用法
    - 加到.gitignore
        - e.g. `/.idea`
    - git rm --cached <檔名>
        - 使用時間:
            - 加入.gitignor前，檔案已經存在，所以會被紀在untracked裡面 因此必須要把cached拿掉
        - e.g. `git rm --cached -r .idea`
            - `-r`: 遞迴，意即將.idea資料夾下的東西都刪掉
            - 'man': 看指令的用法
                -e.g.: `man rm`
            - `-rf`: 與`-r -f`相同，`-f`: force
            
    - filter-branch 用法
        - e.g.:
            - `git filter-branch --tree-filter "rm -rf .idea/*"`
        1. filter-branch 這個指令可以讓你根據不同的 filter，一個一個 Commit 的來處理它。
        2. 這裡使用了 --tree-filter 這個 filter，它可以讓你在 Checkout 到每個 Commit 的時候執行你指定的指令，
        執行完後再自動幫你重新再 Commit。以上面這個例子來說，便是執行「強制刪除 .idea 資料夾」這個指令。
        3. 因為刪除了某個檔案，所以在那之後的 Commit 全部都會重新計算，也就是說等於產生一份新的歷史紀錄了。
    - .gitignore 設為全域
        - 位置: home
        - 檔名: .gitignore_global
            - e.g.
            -   
                ```
                *~
                .DS_Store
                .idea
                ```
                
    - 做完後可 confirm 一下: `git log --stat` 查看是否在 commit 中有把 .gigignore 給忽略掉
    - 參考
        - https://gitbook.tw/chapters/faq/remove-sensitive-data.html
        - https://gitbook.tw/chapters/faq/remove-files-from-git.html
       
- rebase 修改已 commit 的token/password/機密資料
    - 指令: `git rebase -i <指定要開始的位置>`
        - e.g. `git rebase -i --root`
    - 進入互動模式後 把要改的commit設為e (edit) 然後再把code改掉後 rebase --continue
        - 疑點: 可以用`allow-empty`來建一個空的commit，或隨便*touch*一個檔案後*commit* 再用`git rebase -i --root`
        進入互動模式後再把這個新建的*commit*移到最前面 → 用途??
