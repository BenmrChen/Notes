- git 多點合成一點
    1. master git init
    2. master touch .gitignore
    3. master git add .gitignore
    4. master git commit -m 'init'
    5. master git checkout -b Feature
    6. Feature touch 1 2 3 4
    7. Feature git add 1
    8. Feature git commit -m '1'
    9. Feature {repeat step 7. 8. for file 2, 3, 4 }
    10. Feature git branch Dev
    11. Feature git rebase -i --root --onto master Dev
    // 目前在feature分支上，要把1、2、3、4 commit 給合併起來變*4in1*並把Dev分支貼上去，指向master(以master為基底)
    12. {in vim editor} {reword 1, fixup 2, 3, 4}
    13. {in vim editor} {rename commint '4 in 1'}
    14. Dev git checkout master
    15. master git merge Dev
    16. master git checkout Feature
    17. Feature touch 5 6 7 8
    18. Feature git add 5
    19. Feature git commit -m '5'
    20. Feature {repeat step 17. 18. for file 6, 7, 8 }
    21. Feature git branch ToBeMerge
    22. Feature git rebase -i {commit 4} --onto Dev ToBeMerge
    // 目前在feature分支上，要把5、6、7、8 commit 給合併起來變*8in1*並把ToBeMerge分支貼上去，指向Dev(以Dev為基底)
    23. {in vim editor} {reword 5, fixup 6, 7, 8}
    24. {in vim editor} {rename commint '8 in 5'}
    25. ToBeMerge git checkout Dev
    26. Dev git merge ToBeMerge
    ![](https://i.imgur.com/ZezXPPb.jpg)
    https://i.imgur.com/ZezXPPb.jpg

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
