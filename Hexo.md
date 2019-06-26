### npm error EISGIT on npm install
https://github.com/mdbootstrap/bootstrap-material-design/issues/221

### 安裝 Markdown-it 為了正常的 MD 語法
https://soj0825.github.io/2019/01/14/Hexo-Setting/

### Chris 的研究筆記
https://dwatow.github.io/2017/06-18-hexo/re-equip-hexo1/

### Steps for Test & Deploy
1. `hexo clear`
2. `hexo generate` = `hexo g`
3. `hexo server --debug` 本機測試
4. `hexo deploy`
- PS. `hexo g -d` 也可直接generate並deploy
- PSS. 產生文章 `$ hexo new [layout] <title>`        
- PSSS. 提醒：每次部署之前最好把站点文件夹内的 .deploy_git 文件夹删掉，否则可能出现部署之后网站并没有更改。
    https://lhy0609.github.io/2017/08/11/Hexo-hiker%E4%B8%BB%E9%A2%98%E9%85%8D%E7%BD%AE%E4%B8%8E%E7%BE%8E%E5%8C%96/
- 每次写完，进行部署
    - hexo c
    - hexo g -d
    
### 閱讀全文按钮
  ```
  床前明月光
  疑是地上霜
  <!-- more -->
  舉頭望明月
  低頭思故鄉
  ```