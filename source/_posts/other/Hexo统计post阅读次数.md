# Hexo统计post阅读次数

## 说明

有的博主的博客都是自己利用开源的hexo搭建的,对于博客的阅读量的统计不是支持的很好,尤其是我们天朝。
有的主题比如landscape自带google统计,但是天朝你懂的,估计好多人都打不开,翻墙除外。所以我放弃了google,
再者我用的是`TKL`这个主题本来也不支持统计,怎么办呢?后台跑到百度统计,看看能不能搞定,发现api目前并不支持根据url
进行统计提供数据,又放弃了百度。又在[hexo插件](https://hexo.io/plugins/)官网 找个一款`hexo-related-popular-posts`,
看了文档也是基于谷歌分析的,也放弃了。自己搭建一个成本有太高,当要放弃的时候在一篇博客中看到了[leancloud](https://leancloud.cn/)
一个云端存储产品,支持个人和商用,个人有api调用限制,其实对于博客来说基本够了,也就调用set get俩接口。下面奉上实现过程:

[![hexo](http://oq37iabrl.bkt.clouddn.com/image/hexo/hexo1.png)](http://oq37iabrl.bkt.clouddn.com/image/hexo/hexo1.png)

## 注册

登陆[leancloud](https://leancloud.cn/),注册个人帐号

## 新建应用

[![hexo3](http://oq37iabrl.bkt.clouddn.com/image/hexo/hexo3.png)](http://oq37iabrl.bkt.clouddn.com/image/hexo/hexo3.png)

## 设置应用

[![hexo4](http://oq37iabrl.bkt.clouddn.com/image/hexo/hexo4.png)](http://oq37iabrl.bkt.clouddn.com/image/hexo/hexo4.png)

appid和appkey
[![hexo5](http://oq37iabrl.bkt.clouddn.com/image/hexo/hexo5.png)](http://oq37iabrl.bkt.clouddn.com/image/hexo/hexo5.png)

设置安全域名
支持多个,也可以填写ip地址

[![hexo6](http://oq37iabrl.bkt.clouddn.com/image/hexo/hexo6.png)](http://oq37iabrl.bkt.clouddn.com/image/hexo/hexo6.png)

## 创建API类

[![hexo7](http://oq37iabrl.bkt.clouddn.com/image/hexo/hexo7.png)](http://oq37iabrl.bkt.clouddn.com/image/hexo/hexo7.png)

这里我取名为Counter

至此,在leancloud 里的设置我们算是结束了。

## 项目引用

由于hexo是node javascript版本,所以我们打开官方javascript [sdk安装说明](https://leancloud.cn/docs/sdk_setup-js.html),按照步骤来

- 配置config.yml
  打开项目根目录,编辑`config.yml`,加入以下编码,app_id 和app_key再上面的设置里可以获取到

```
# add post views
leancloud_visitors:
  enable: true
  app_id: xxxxx
  app_key: xxxxx
```

- 设置html

在casper/post.ejs中添加如下代码

```
<% if(config.leancloud_visitors.enable){ %>阅读数:
    <span id="<%= url_for(post.path) %>" class="leancloud_visitors" data-flag-title="<%- post.title %>"></span>次
<% } %>
```

- 引用js

在footer.ejs,里我们添加如下代码

```
<script src="//cdn1.lncld.net/static/js/2.5.0/av-min.js"></script>
<script>
    var APP_ID = '<%- config.leancloud_visitors.app_id %>';
    var APP_KEY = '<%- config.leancloud_visitors.app_key %>';
    AV.init({
        appId: APP_ID,
        appKey: APP_KEY
    });
    // 显示次数
    function showTime(Counter) {
        var query = new AV.Query("Counter");
        if($(".leancloud_visitors").length > 0){
            var url = $(".leancloud_visitors").attr('id').trim();
            // where field
            query.equalTo("words", url);
            // count 
            query.count().then(function (number) {
                // There are number instances of MyClass where words equals url.
                $(document.getElementById(url)).text(number?  number : '--');
            }, function (error) {
                // error is an instance of AVError.
            });
        }
    }
   // 追加pv
    function addCount(Counter) {
        var url = $(".leancloud_visitors").length > 0 ? $(".leancloud_visitors").attr('id').trim() : 'icafebolger.com';
        var Counter = AV.Object.extend("Counter");
        var query = new Counter;
        query.save({
            words: url
        }).then(function (object) {

        })
    }
    $(function () {
        var Counter = AV.Object.extend("Counter");
        addCount(Counter);
        showTime(Counter);
    });
</script>
```

- 最终效果

[![hexo9](http://oq37iabrl.bkt.clouddn.com/image/hexo/hexo9.png)](http://oq37iabrl.bkt.clouddn.com/image/hexo/hexo9.png)