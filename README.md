# 项目代号：土拨鼠（tubo）

[前言](doc/pre.md)

![土拨](tubo.png)

用途： 微信开发/API对接/自动化测试/抢票脚本/网站监控/点赞插件/数据爬取

Smile Tip: Chinese is easy to learn, you are smart enough! Show me the code. Don't sat any thing, Ahaha~

知乎全能小工具示例正在开发中: [zhihuxx](https://github.com/hunterhug/zhihuxx)，入门必备！

## 一.下载

自己封装的爬虫库,类似于Python的requests,你只需通过该方式获取库

```
go get -v github.com/hunterhug/GoSpider
```

或者新建 你的GOPATH路径/src/github.com/hunterhug

```
cd src/github.com/hunterhug
git clone https://github.com/hunterhug/GoSpider
```

默认所有第三方库已经保存在vendor,如果使用包冲突了,请把vendor下的包移到GOPATH下,谨记！！！GOPATH文件夹下的包为不适宜放在vendor下,请手动移动


以下可选,vendor中已经带全第三方库,使用Go1.8

```
godep restore
```

文件目录（组件化开发）

```
    ---example   爬虫示例,新爬虫已经转移到新仓库
    ---query     内容解析库,只封装了两个方法
    ---spider    爬虫核心库
    ---store     存储库
        ---myredis 
        ---mysql
        ---myetcd
        ---mydb  关系型数据库Orm(使用xorm)
        ---myhbase
        ---mycassandra
    ---util      杂项工具
        --- image 图片切割库
    ---vendor    第三方依赖包
    ---GOPATH    不宜放在vendor的包
```

## 二.使用

HelloWorld Simple一般情况,看代码注释。

```go
package main

import (
	// 第一步：引入库 别名boss
	boss "github.com/hunterhug/GoSpider/spider"
	//"github.com/hunterhug/GoSpider/util"
)

func init() {
	// 第二步：可选设置全局
	boss.SetLogLevel(boss.DEBUG) // 设置全局爬虫日志,可不设置,设置debug可打印出http请求轨迹
	boss.SetGlobalTimeout(3)     // 爬虫超时时间,可不设置

}
func main() {

	log := boss.Log() // 爬虫为你提供的日志工具,可不用

	// 第三步： 必须新建一个爬虫对象
	//spiders, err := boss.NewSpider("http://smart:smart2016@104.128.121.46:808") // 代理IP爬虫 格式:协议://代理帐号(可选):代理密码(可选)@ip:port
	//spiders, err := boss.NewSpider(nil)  // 正常爬虫 默认带Cookie
	//spiders, err := boss.NewAPI() // API爬虫 默认不带Cookie
	spiders, err := boss.New(nil) // NewSpider同名函数
	if err != nil {
		panic(err)
	}

	// 第四步：设置抓取方式和网站,可链式结构设置,只有SetUrl是必须的
	// SetUrl:Url必须设置
	// SetMethod:HTTP方法可以是POST或GET,可不设置,默认GET,传错值默认为GET
	// SetWaitTime:暂停时间,可不设置,默认不暂停
	spiders.SetUrl("http://www.google.com").SetMethod(boss.GET).SetWaitTime(2)
	spiders.SetUa(boss.RandomUa())                 //设置随机浏览器标志
	spiders.SetRefer("http://www.google.com")      // 设置Refer头
	spiders.SetHeaderParm("diyheader", "lenggirl") // 自定义头部

	//spiders.SetBData([]byte("file data")) // 如果你要提交JSON数据/上传文件
	//spiders.SetFormParm("username","jinhan") // 提交表单
	//spiders.SetFormParm("password","123")

	// 第五步：开始爬
	//spiders.Get()             // 默认GET
	//spiders.Post()            // POST表单请求,数据在SetFormParm()
	//spiders.PostJSON()        // 提交JSON请求,数据在SetBData()
	//spiders.PostXML()         // 提交XML请求,数据在SetBData()
	//spiders.PostFILE()        // 提交文件上传请求,数据在SetBData()
	body, err := spiders.Go() // 如果设置SetMethod(),采用,否则Get()
	if err != nil {
		log.Error(err.Error())
	} else {
		//log.Infof("%s", string(spiders.Raw)) // 打印获取的数据
		log.Infof("%s", string(body)) // 打印获取的数据

		//util.JsonBack(body) // 如果获取到的是JSON数据,转义回来,不然会乱码
	}

	log.Debugf("%#v", spiders) // 不设置全局log为debug是不会出现这个东西的

	//spiders.ClearAll() // 爬取完毕后可以清除设置的Http头部和POST的表单数据/文件数据/JSON数据
	spiders.Clear() // 爬取完毕后可以清除POST的表单数据/文件数据/JSON数据

	// 爬虫池子
	boss.Pool.Set("myfirstspider", spiders)
	if poolspider, ok := boss.Pool.Get("myfirstspider"); ok {
		poolspider.SetUrl("http://www.baidu.com")
		data, _ := poolspider.Get()
		log.Info(string(data))
	}
}
```

使用特别简单,先`New`一只`Spider`,然后`SetUrl`,适当加头部,最后`spiders.Go()`即可。

### 第一步

爬虫有三种类型:

1. `spiders, err := boss.NewSpider("http://smart:smart2016@104.128.121.46:808") ` // 代理IP爬虫 格式:`协议://代理帐号(可选):代理密码(可选)@ip:port` 别名函数`New()`
2. `spiders, err := boss.NewSpider(nil)`  // 正常爬虫 默认带Cookie 别名函数`New()`
3. `spiders, err := boss.NewAPI()` // API爬虫 默认不带Cookie

### 第二步

模拟爬虫设置头部:

1. `spiders.SetUrl("http://www.lenggirl.com")`  // 设置Http请求要抓取的网址,必须
2. `spiders.SetMethod(boss.GET)`  // 设置Http请求的方法:`POST/GET/PUT/POSTJSON`等
3. `spiders.SetWaitTime(2)` // 设置Http请求超时时间
4. `spiders.SetUa(boss.RandomUa())`                // 设置Http请求浏览器标志,本项目提供445个浏览器标志，可选设置
5. `spiders.SetRefer("http://www.baidu.com")`       // 设置Http请求Refer头
6. `spiders.SetHeaderParm("diyheader", "lenggirl")` // 设置Http请求自定义头部
7. `spiders.SetBData([]byte("file data"))` // Http请求需要上传数据
8. `spiders.SetFormParm("username","jinhan")` // Http请求需要提交表单

更多自行查看源代码(高级)

### 第三步

爬虫启动方式有：
1. `body, err := spiders.Go()` // 如果设置SetMethod(),采用,否则Get()
2. `body, err := spiders.Post()` // POST表单请求,数据在SetFormParm()
3. `body, err := spiders.Get()` // 默认
4. `body, err := spiders.PostJSON()` // 提交JSON请求,数据在SetBData()
5. `body, err := spiders.PostXML()` // 提交XML请求,数据在SetBData()
6. `body, err := spiders.PostFILE()` // 提交文件上传请求,数据在SetBData()

### 第四步

爬取到的数据：

1. `body, err := spiders.Go() log.Infof("%s", string(body))` // 默认
2. `log.Infof("%s", string(spiders.Raw))` // 打印获取的数据,数据在http响应后会保存在Raw中
3. `body, err := spiders.Go() util.JsonBack(body)` // 如果获取到的是JSON数据,转义回来,不然会乱码

注意：每次抓取网站后,下次请求你可以覆盖原先的头部,但是没覆盖的头部还是上次的,所以清除头部或请求数据,请使用`Clear()`(只清除Post数据)或者`ClearAll()`(还清除http头部)

更多用法：如多只爬虫并发,使用爬虫池子,`boss.Pool.Set("myfirstspider", spiders)`,参见[分布式文章爬取](http://www.lenggirl.com/spider/jiandan.html)

[API参考](doc/api.md)

## 三.具体例子
### 1.入门

a.见[helloworld](example/helloworld/README.md)

b.见[图片下载](example/taobao/README.md)

### 2.示例项目

高级示例项目见[http://www.github.com/hunterhug/GoSpiderExample](http://www.github.com/hunterhug/GoSpiderExample)


如果你觉得项目帮助到你,欢迎请我喝杯咖啡

微信
![微信](https://raw.githubusercontent.com/hunterhug/hunterhug.github.io/master/static/jpg/wei.png)

支付宝
![支付宝](https://raw.githubusercontent.com/hunterhug/hunterhug.github.io/master/static/jpg/ali.png)

版本日志信息见[日志](doc/log.md)

## 五.环境配置

### Go安装

a. Ubuntu安装

[云盘](https://yun.baidu.com/s/1jHKUGZG)下载源码解压.下载IDE也是解压设置环境变量.

```
vim /etc/profile.d/myenv.sh

export GOROOT=/app/go
export GOPATH=/home/jinhan/code
export GOBIN=$GOROOT/bin
export PATH=.:$PATH:/app/go/bin:$GOPATH/bin:/home/jinhan/software/Gogland-171.3780.106/bin

source /etc/profile.d/myenv.sh
```

b. Windows安装

[云盘](https://yun.baidu.com/s/1jHKUGZG) 选择后缀为msi安装如1.6

环境变量设置：

```
Path G:\smartdogo\bin
GOBIN G:\smartdogo\bin
GOPATH G:\smartdogo
GOROOT C:\Go\
```

### MYSQL安装

a. Ubuntu安装

敲入以下命令按提示操作
```
sudo apt-get install mysql-server mysql-client
```

开关

```
开启  ./mysqld_safe &

关闭  mysqladmin -uroot shutdown
```

b. Windows安装

[https://yun.baidu.com/s/1hrF0QC8](https://yun.baidu.com/s/1hrF0QC8) 找到mysql文件夹下面的5.6.17.0.msi根据说明安装.

## 六.高级配置（docker）

我们的库可能要使用各种各样的工具，配置连我这种专业人员有时都搞不定，而且还可能会损坏，所以用docker方式随时随地开发。

`Golang,Mysql,Redis`都可以通过这种方式快速搭起来，然后你在容器内部进行操作，我们的容器启动都采用一次性容器`--rm`,不怕你在后台跑后忘了，同时网络模式都是`--net=host`,共用主机端口和IP。

先拉镜像

```
docker pull golang:1.8
```

Golang环境启动：

```
docker run --rm --net=host -it -v /home/jinhan/code:/go --name mygolang golang:1.8 /bin/bash

root@27214c6216f5:/go# go env
GOARCH="amd64"
```

其中`/home/jinhan/code`为你自己的本地文件夹（虚拟GOPATH），你在docker内`go get`产生在`/go`的文件会保留在这里，容器死掉，你的`/home/jinhan/code`还在，你可以随时修改文件配置。

启动后你就可以在里面开发了。

什么？你还需要Redis?

拉：

```
docker pull redis:3.2
```

如果需要Redis服务器,请启动：

```
docker run --rm --net=host --name="myredis-server" -p 6379:6379 -v /home/jinhan/redis/data:/data  -it redis:3.2 redis-server --appendonly yes
```

- `-p 6379:6379` :将容器的6379端口映射到主机的6379端口，使用host模式其实是不用的
- `-v /home/jinhan/redis/data` :将主机中/home/jinhan/redis/data文件夹挂载到容器的/data
- `-it`: 表示前台跑，如果想后台，请`-d`，建议不要
- `redis-server --appendonly yes` :在容器执行redis-server启动命令，并打开redis持久化配置

使用Redis客户端访问：

```
docker run --rm --net=host --name="myredis-client" -it redis:3.2 redis-cli
```

什么？你还需要Mysql?这个太难配置，请装本地。

## 七.备注
		
此库采用[Glide](https://github.com/Masterminds/glide)方式管理第三方库（使用者可以忽略,中国防火长城让我爪机,最终完全弃用,长城太猛）		
	
```		
$ glide init                              # 创建工作区		
$ open glide.yaml                         # 编辑glide.yaml文件		
$ glide get github.com/hunterhug/GoSpider # get下库然后会自动写入glide.yaml	
	
	
$ glide install                           # 安装,没有glide.lock,会先运行glide up
$ go build                                # 试试可不可以跑		
$ glide up                                # 更新库,创建glide.lock		
```

改用[Godep](https://github.com/tools/godep) 长城依旧太猛
 
```
godep save
godep update -goversion
godep restore
```
# LICENSE

欢迎加功能(PR/issues),请遵循Apache License协议(即可随意使用但每个文件下都需加此申明）

```
Copyright 2017 by GoSpider author.
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
    http://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License
```
