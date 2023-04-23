centos7抓取grafana图片

## 1、背景

想要一个CPU的使用率的日报，因此想程序直接抓取grafana的Panel生成的图片，这样不用每次都打开grafana，然后手动截图，再保存，再发日报。

谷歌了一下，发现官方有个插件叫grafana-image-renderer，能直接渲染图片，生成图片链接，只需要python爬虫抓取该链接，保存图片即可，后续写个发邮件的程序就行。

(谷歌发现两种办法，另一种是使用grafana-reporter，因为这个要起一个Go新服务，因此我没有尝试该办法，需要的小伙伴自行谷歌吧)

## 2、步骤

首先说明一点，我的grafana服务是本地安装，不是docker部署

### 2.0、未安装插件时的表现

未安装grafana-image-renderer插件时，点击share按钮分享panel，会提示图片渲染插件未安装，要求安装一下

![image-20221019143705819](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221019143705819.png)

### 2.1、首先要安装grafana-image-renderer插件

官网安装指导：https://grafana.com/grafana/plugins/grafana-image-renderer/

官网配置文件指导：https://grafana.com/docs/grafana/v9.0/setup-grafana/image-rendering/#configuration

执行命令安装插件

```sh
grafana-cli plugins install grafana-image-renderer
```

![image-20221018170736932](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221018170736932.png)

### 2.2、安装好后重启一下grafana服务

```sh
systemctl restart grafana-server.service
```

安装好后可以查看一下是否有该插件了

```sh
grafana-cli plugins ls
```

![image-20221018170856313](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221018170856313.png)

### 2.3、该插件的底层原理应该是调用chrome无头浏览器进行操作，因此我们需要看看是不是依赖都全面

```sh
ldd /var/lib/grafana/plugins/grafana-image-renderer/chrome-linux/chrome
```

![image-20221018170928769](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221018170928769.png)

我们需要把not found的依赖都安装上，这里可以直接去https://pkgs.org/网址找到依赖，然后安装

例子：

（1）提示缺少libatk-1.0.so.0，则我们去https://pkgs.org/，输入框输入libatk-1.0.so.0，然后搜索，找到系统对应的软件包，例如我的是centos7，然后点击进去

![image-20221018172259191](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221018172259191.png)

点击进去以后，可以看到提示用`yum install atk`下载即可，

![image-20221018172408471](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221018172408471.png)

例子2：缺libXcomposite.so.1文件，也是类似例子1一样，

![image-20221018172855346](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221018172855346.png)

![image-20221018172917998](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221018172917998.png)

这个缺少依赖的问题，也可以在官网的解决重大问题的链接看到：https://grafana.com/docs/grafana/v9.0/setup-grafana/image-rendering/troubleshooting/

![image-20221019165238972](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221019165238972.png)

我解决该问题，安装了这些包

![image-20221018173343352](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221018173343352.png)

![image-20221018173426252](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221018173426252.png)

官网也提到centos7的依赖包，其他系统看官网就行

```bash
libXcomposite libXdamage libXtst cups libXScrnSaver pango atk adwaita-cursor-theme adwaita-icon-theme at at-spi2-atk at-spi2-core cairo-gobject colord-libs dconf desktop-file-utils ed emacs-filesystem gdk-pixbuf2 glib-networking gnutls gsettings-desktop-schemas gtk-update-icon-cache gtk3 hicolor-icon-theme jasper-libs json-glib libappindicator-gtk3 libdbusmenu libdbusmenu-gtk3 libepoxy liberation-fonts liberation-narrow-fonts liberation-sans-fonts liberation-serif-fonts libgusb libindicator-gtk3 libmodman libproxy libsoup libwayland-cursor libwayland-egl libxkbcommon m4 mailx nettle patch psmisc redhat-lsb-core redhat-lsb-submod-security rest spax time trousers xdg-utils xkeyboard-config alsa-lib
```

## 3、验证

安装插件成功，安装依赖成功，重启grafana，点击页面分享，就可以看到可以渲染图片了（**Direct link rendered image**）

![image-20221019165608106](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221019165608106.png)

图片默认保存位置是grafana的data目录下的png目录，保存天数默认为1天，默认位置是/var/lib/grafana/png

![image-20221019170108495](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221019170108495.png)

## 4、常见问题

（1）最常见的问题应该就是缺失依赖包，从官网可以解决，其他常见问题也可以直接看官网

https://grafana.com/docs/grafana/v9.0/setup-grafana/image-rendering/troubleshooting/

当然还可以看github的issue

github官网：https://github.com/grafana/grafana-image-renderer

（2）我遇到一个问题，提示是**code = Unauthenticated desc = Unauthorized request"**

这是 /var/log/grafana/grafana.log的日志

level=error msg="Rendering failed." error="rpc error: code = Unauthenticated desc = Unauthorized request"

![image-20221019152917727](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221019152917727.png)

从官网的issue得到了解决答案,原来是下载匹配版本就行，我下载的是version=9.2.1

https://github.com/grafana/grafana-image-renderer/issues/370

![image-20221019171112515](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221019171112515.png)

注：每个版本都有对应匹配的，参考官网https://grafana.com/grafana/plugins/grafana-image-renderer/?tab=installation

![image-20221027095811677](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221027095811677.png)

## 5、python爬虫请求

（1）如果response.text中出现**If you're seeing this Grafana has failed to load its application files**，则可能是需要验证，可以去grafana生成一个APIkey

**错误信息**

**If you're seeing this Grafana has failed to load its application files**

\1. This could be caused by your reverse proxy settings.

\2. If you host grafana under subpath make sure your grafana.ini root_url setting includes subpath. If not using a reverse proxy make sure to set serve_from_sub_path to true.

\3. If you have a local dev build make sure you build frontend using: yarn start, yarn start:hot, or yarn build

\4. Sometimes restarting grafana-server can help

\5. Check if you are using a non-supported browser. For more information, refer to the list of [supported browsers](https://grafana.com/docs/grafana/latest/installation/requirements/#supported-web-browsers).

**错误页面**(把response.text写到html后展示)

![image-20221026112231961](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221026112231961.png)

（2）如果/var/log/grafana/grafana.log日志中出现302，则也可能是需要认证

logger=context userId=0 orgId=0 uname= t=2022-10-26T10:58:40.245237482+08:00 level=info msg="Request Completed" method=GET path=/render/d-solo/nano-vm-detail/nanoxu-ni-ji-ming-xi **status=302** remote_addr=172.16.70.189 time_ms=1 duration=1.300399ms size=29 referer= handler=/render/*

（3）如果出现401，则也是需要认证

API keys生成页面

![image-20221019171153866](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221019171153866.png)

然后请求的时候加在header中就行，保存png图片即可

```python
import requests

response=requests.get("http://172.16.90.18:3000/render/d-solo/shEoIlI4k/cpushi-yong-lu?orgId=1&from=1666142237595&to=1666163837596&panelId=2&width=1000&height=500&tz=Asia%2FShanghai",headers={"Authorization":"Bearer sdfsdfsd=="})
print(response.status_code)
filename='show.png'
with open(filename, 'wb') as f:
    f.write(response.content)
print(f"download png success, save to {filename}")
```

## 6、支持中文标题的图片

默认是不支持中文的，可以通过配置来改变，配置参考上面提到的官网链接就行https://grafana.com/docs/grafana/v9.0/setup-grafana/image-rendering/#configuration

![image-20221019171540105](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221019171540105.png)

配置文件修改/etc/grafana/grafana.ini

```ini
[plugin.grafana-image-renderer] 
rendering_timezone = Asia/Shanghai 
rendering_language = zh-CN,zh;q=0.9
```

然后安装中文字体

centos7执行`yum groupinstall Fonts` 其他机器还需要谷歌一下

参考：https://github.com/grafana/grafana/issues/24729

![image-20221019190724911](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221019190724911.png)

重启

```sh
systemctl restart grafana-server.service
```

可以看到已经支持中文了

![image-20221019190808595](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221019190808595.png)

docker方式参考链接是：https://blog.csdn.net/dandanfengyun/article/details/115346594

https://blog.csdn.net/lee_yanyi/article/details/120364080