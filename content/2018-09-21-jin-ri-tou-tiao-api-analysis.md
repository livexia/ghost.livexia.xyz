+++
title = "今日头条爬取分析"
slug = "jin-ri-tou-tiao-api-analysis"
date = 2018-10-15T06:12:00.000Z
excerpt = "课程需要对今日头条的内容解析进行一个分析"

[taxonomies]
tags = ["Coding"]
+++

### 课程需要对今日头条的内容解析进行一个分析

存在api：[今日头条Api分析](https://github.com/iMeiji/Toutiao/wiki/%E4%BB%8A%E6%97%A5%E5%A4%B4%E6%9D%A1Api%E5%88%86%E6%9E%90)

正文：

评论：

1. PC端可以获得20条评论

2. 其余需要手机端api

分析：

评论存在Api（PC\手机）

# Update

    url="https://www.toutiao.com/api/pc/feed/"
    params={
        "category":"news_hot",
        "utm_source":"toutiao",
        "widen":"1",
        "max_behot_time":"1539572090",
        "max_behot_time_tmp":"1539572090",
        "tadrequire":"true",
        "as":"A1E54BFCD480436",
        "cp":"5BC4705493D6EE1",
        "_signature":"PzN5IAAAZP2VdNF1CwwtFj8zeT",
    }
    headers={
        "user-agent":"Mozilla/5.0(Macintosh;IntelMacOSX10_14_0)AppleWebKit/537.36(KHTML,likeGecko)"
        "Chrome/69.0.3497.100Safari/537.36"
    }
    

重点关键字：

category：类别

max_behot_time：时间（难以获取，由服务器给出算法未知）

_signature：关键签名，代表唯一认证

结果：无法利用该入口获取所有新闻，尝试更换api

相比于正文，评论的api更好获取，尝试利用搜索关键字获取新闻列表

    url="https://www.toutiao.com/search_content/"
    offset=20
    count=20
    keyword="鸿茅药酒"
    params={
        "offset":offset,
        "format":"json",
        "keyword":keyword,
        "count":count
    }
    headers={
        "user-agent":"Mozilla/5.0(Macintosh;IntelMacOSX10_14_0)AppleWebKit/537.36(KHTML,likeGecko)"
        "Chrome/69.0.3497.100Safari/537.36"
    }
    

结果：可行

利用上面的搜索结果api取得新闻id

新闻正文api：[https://m.toutiao.com/i6612377651942261256/info/](https://m.toutiao.com/i6612377651942261256/info/)

新闻评论api：[https://ic.snssdk.com/article/v2/tab_comments/?count=50&group_id=6612377651942261256&item_id=6612377651942261256&offset=200](https://ic.snssdk.com/article/v2/tab_comments/?count=50&amp;group_id=6612377651942261256&amp;item_id=6612377651942261256&amp;offset=200)
