# personal-hugo-blog

personal blog by using hugo

## 配置自己的留言系统
> 修改后这连个属于git字模块,不用提交,以便以后更新这个主题最新版,使用新版功能
```shell
修改 ${blog_site}/themes/hugo-clarity/layouts/partials/comments.html 内容如下
```

```javascript
<!--<div class="post_comments">-->
<!--  {{ if .Site.DisqusShortname }}-->
<!--    {{ template "_internal/disqus.html" . }}-->
<!--  {{ end }}-->
<!--  {{ if .Site.Params.utterances }}-->
<!--    {{ template "partials/utterances.html" . }}-->
<!--  {{ end }}-->
<!--  &lt;!&ndash; add custom comments markup here &ndash;&gt;-->
<!--</div>-->
<div class="post_comments">
  {{ template "partials/remark42.html" . }}
</div>
```
同目录新建remark42.html,内容如下:
```javascript
<script>
    var remark_config = {
        host: "https://remark42.blentle.com", // hostname of remark server, same as REMARK_URL in backend config, e.g. "https://demo.remark42.com"
        site_id: 'blentle-blog',
        components: ['embed'], // optional param; which components to load. default to ["embed"]
                               // to load all components define components as ['embed', 'last-comments', 'counter']
                               // available component are:
                               //     - 'embed': basic comments widget
                               //     - 'last-comments': last comments widget, see `Last Comments` section below
                               //     - 'counter': counter widget, see `Counter` section below
        url: window.location.origin + window.location.pathname, // optional param; if it isn't defined
                         // `window.location.origin + window.location.pathname` will be used,
                         //
                         // Note that if you use query parameters as significant part of url
                         // (the one that actually changes content on page)
                         // you will have to configure url manually to keep query params, as
                         // `window.location.origin + window.location.pathname` doesn't contain query params and
                         // hash. For example default url for `https://example/com/example-post?id=1#hash`
                         // would be `https://example/com/example-post`.
                         //
                         // The problem with query params is that they often contain useless params added by
                         // various trackers (utm params) and doesn't have defined order, so Remark treats differently
                         // all this examples:
                         // https://example.com/?postid=1&date=2007-02-11
                         // https://example.com/?date=2007-02-11&postid=1
                         // https://example.com/?date=2007-02-11&postid=1&utm_source=google
                         //
                         // If you deal with query parameters make sure you pass only significant part of it
                         // in well defined order
        max_shown_comments: 100, // optional param; if it isn't defined default value (15) will be used
        theme: 'light', // optional param; if it isn't defined default value ('light') will be used
        page_title: 'Moving to Remark42', // optional param; if it isn't defined `document.title` will be used
        locale: 'zh', // set up locale and language, if it isn't defined default value ('en') will be used
        show_email_subscription: false // optional param; by default it is `true` and you can see email subscription feature
                                       // in interface when enable it from backend side
                                       // if you set this param in `false` you will get notifications email notifications as admin
                                       // but your users won't have interface for subscription
    };
</script>
<script>
    (function(c) {
        for(var i = 0; i < c.length; i++){
            var d = document, s = d.createElement('script');
            s.src = remark_config.host + '/web/' +c[i] +'.js';
            s.defer = true;
            (d.head || d.body).appendChild(s);
        }
    })(remark_config.components || ['embed']);
</script>
<div id="remark42"></div>
```

## debug
+ debug without draft
```shell
  hugo server  
```


+ debug with draft
 ```shell
   hugo server -D 
 ```

## publish
+ publish to pure static website
```shell
    hugo
```

## New post
+ create a new post
```shell
    hugo new --kind post post/xxx.md 
```