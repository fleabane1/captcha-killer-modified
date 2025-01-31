* [验证码二次处理案例](#6-验证码进行二次处理的案例验证码为gif图且验证码具体是在gif图的第二帧无法直接识别)
* [验证码调试方法](7-验证码识别顺序错误验证码识别正确但是intruder登录接口的验证码识别错误的调试方法)
* [验证码响应包有token参数，登录需校验情况如何设置](8-验证码中有token等校验参数返回登录包中有校验token如何进行设置爆破)

# 有问题请在FAQ或者README寻找一下，如果没找到请提issue
# 1-用法

#### [releases](https://github.com/f0ng/captcha-killer-modified/releases/)下载最新插件与验证码识别端(`captcha-killer-modified.jar`、`codereg.py`)
#### 使用Burp加载`captcha-killer-modified.jar`
#### 再使用`python3 codereg.py`开启验证码识别模块，前提安装[ddddocr](https://github.com/sml2h3/ddddocr)
#### <a id="Template">模板</a>
```
POST /reg HTTP/1.1
Host: 127.0.0.1:8888
Authorization:Basic f0ngauth
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:97.0) Gecko/20100101 Firefox/97.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 8332

<@BASE64><@IMG_RAW></@IMG_RAW></@BASE64>
```
#### 接口地址设置为`http://127.0.0.1:8888`，即可开始识别
#### 其余用法移步[captcha-killer用法](https://gv7.me/articles/2019/burp-captcha-killer-usage/)

# 2-加载插件错误

#### 0x01 检查加载的是否是jar文件
#### 0x02 与启动burp的jdk版本有关，可以适当选择与`jdk10`相近的版本启动

# 3-验证码识别错误

#### 1.查看模板是否正确,<a href="#Template">模板</a>
#### 2.检查`python3 codereg.py`是否正常启动，可以本地尝试使用ddddocr进行识别验证码验证
#### 3.重新下载本插件与验证码服识别端进行加载

# 4.报错 `Typeerror: classification() got an unexpected keyword argument img_base64`

#### 4 感谢ekko-zhao师傅反馈
#### 修改`codereg.py`源码如下（记得`import base64`）
```python
async def handle_cb(request) :
    img_base64 = await request.text()
    img_bytes = base64.b64decode(img_ base64)
    return web. Response(text=ocr.classification(img_bytes))
```


# 5-无法识别验证码

#### 一般问题出在新版burp上，由于jdk的原因导致代码运行错误，可以下载相应jdk版本的插件

<img width="1234" alt="image" src="https://user-images.githubusercontent.com/48286013/171561756-2a74a18b-8ad0-47f8-a17a-309b98784046.png">

# 6-验证码进行二次处理的案例(验证码为gif图,且验证码具体是在gif图的第二帧,无法直接识别)

可以自写中转，将处理后的验证码通过接口返回，如下代码:
```python
# -*- coding: utf-8 -*-  
# @Time    : 2022/11/29 4:22 下午  
# @Software: f0ng  
from flask import Flask,Response,render_template  
  
app = Flask(__name__)  
  
import requests,time  
  
proxies = {  
'http':'127.0.0.1:8080',  
'https':'127.0.0.1:8080'  
}  
  
def shibie():  
    headers = {  # 校验用户信息
        "Cookie":"ASP.NET_SessionId=fmbv2azmygla0hfkt3v5dupt",  
    }  
  
    url = 'http://xxxxx.xxxx/CreateCode' # 目标下载链接  
    r = requests.get(url ,headers=headers ,proxies=proxies) # 发送请求  
    with open ('1.gif', 'wb+') as f:  
        f.write(r.content)  
        f.close  
  
    from PIL import Image, ImageFile  
    ImageFile.LOAD_TRUNCATED_IMAGES = True  
  
    import os  
    gifFileName = '1.gif'  
    # 使用Image模块的open()方法打开gif动态图像时，默认是第一帧  
    im = Image.open(gifFileName)  
    pngDir = gifFileName[:-4]  
    # 创建存放每帧图片的文件夹(文件夹名与图片名称相同)  
    if os.path.exists(pngDir) == False:  
        os.mkdir(pngDir)  
    try:  
        while True:  
            # 保存当前帧图片  
            current = im.tell()  
            im.save(pngDir + '/' + str(current) + '.png')  
            # 获取下一帧图片  
            im.seek(current + 1)  
    except EOFError:  
        print("pass")  
  
@app.route("/1")  
def index3():  
    shibie()  
  
    with open("1/0.png", 'rb') as f:  
        image = f.read()  
    resp = Response(image, mimetype="image/jpeg")  
    return resp  
  
if __name__ == '__main__':  
    app.debug = True  # 设置调试模式，生产模式的时候要关掉debug  
    app.run(host="0.0.0.0", port="8888")
```

代码先将gif截取成1.png，再通过接口`/1`返回1.png的图片，故获取验证码的请求可以改为本地的`/1`请求，如下图

<img width="561" alt="image" src="https://user-images.githubusercontent.com/48286013/204831775-12883d4f-86cb-4ac7-8aa2-1ee9dedbf4b8.png">

实现成功

<img width="527" alt="image" src="https://user-images.githubusercontent.com/48286013/204831853-2c1d4773-f31d-42e3-92ae-340ccb377dfc.png">

# 7-验证码识别顺序错误(验证码识别正确，但是intruder登录接口的验证码识别错误)的调试方法

插件里获取验证码->识别->手动在repeater输入->显示验证码是否正确

如果验证码不正确，说明请求验证码的cookie等校验用户信息不一致，需要加上该cookie等校验用户信息的参数，如果为二次处理验证码的请求，需在请求验证码的代码中带上用户cookie等校验用户信息的参数。

# 8-验证码中有token等校验参数返回，登录包中有校验token，如何进行设置爆破？

输入响应提取的正则，提取校验的关键字

<img width="650" alt="image" src="https://user-images.githubusercontent.com/48286013/204822669-7ea6022e-8028-4526-a653-03488a196d48.png">

在intruder中增加校验的参数`@captcha-killer-modified@`

<img width="573" alt="image" src="https://user-images.githubusercontent.com/48286013/204827078-dddbbd99-9c96-4c77-9542-20c5a5d1b033.png">

在logger或者logger++中可以看到实际的请求

<img width="638" alt="image" src="https://user-images.githubusercontent.com/48286013/204827499-35424d0e-7070-4905-ac4f-46f05a15d49e.png">
