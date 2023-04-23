npm install 报错提示github需要权限 Permission denied (publickey)

### 1、报错信息

npm ERR! code 128
npm ERR! An unknown git error occurred
npm ERR! command git --no-replace-objects ls-remote ssh://git@github.com/layouts/Admin.js.git
npm ERR! git@github.com: Permission denied (publickey).
npm ERR! fatal: Could not read from remote repository.
npm ERR!
npm ERR! Please make sure you have the correct access rights
npm ERR! and the repository exists.

npm ERR! A complete log of this run can be found in:
npm ERR!     C:\Users\dell\AppData\Local\npm-cache\_logs\2022-05-30T05_33_24_378Z-debug-0.log

截图：

![1653889264816](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1653889264816.png)

### 2、解决办法

运行以下命令

```bash
git config --global http.sslverify "false"
```

然后再执行

```bash
npm install
```

 参考 ： https://blog.csdn.net/m0_51969330/article/details/118895837