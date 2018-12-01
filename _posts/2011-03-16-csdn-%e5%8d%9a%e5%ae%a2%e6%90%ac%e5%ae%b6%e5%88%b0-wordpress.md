---
id: 250
title: CSDN 博客搬家到 WordPress
date: 2011-03-16T22:51:33+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=250
permalink: /csdn-%e5%8d%9a%e5%ae%a2%e6%90%ac%e5%ae%b6%e5%88%b0-wordpress/
tags:
  - console
  - log
  - blog_backup
  - csdn
  - export
  - import
  - wordpress
---

在 CSDN 上断断续续写博客也有好几年了，但 CSDN 博客还是那个三俗的样子。终于，打算换地方了，并且打算好好写写。第一步要做的就是博客搬家。这里就要先数落一下CSDN博客了，没有一个导出博客的功能，MSN Space 搬家，点击几下鼠标就搞定，但CSDN博客，我还得写这么一篇帖子来记录我是如何搬家的。

先在网上搜索一番，发现了这么个工具 <a href="http://www.pt42.cn/blog_backup_index.htm">blog_backup</a>, 是一个收费软件，试用版可以把博客文章导到本地，如果要导出，比如 wordpress 格式，则要收费，这里要感慨一下作者的商业头脑。点击注册页面居然进入了淘宝店铺，一个注册码¥40。当然，作者也不容易，这个软件一直在维护升级。

闲话不说，既然它把文章导到了本地，那就有办法可想了。看看它的目录，发现了一个 myblog.db 文件，用记事本打开，赫然写着 sqlite3，原来是用 sqlite 保存的。去 sqlite.org 下载一个 console program， 先瞅瞅 db 文件里有些啥。 下完才发现 Mac OSX 自带了 sqlite3， 所以下面的命令行是在 OSX 下的, 但稍加修改应该就可以在 windows 下用了。

	nickx:~/Downloads $ sqlite3 myblog.sqlite
	SQLite version 3.6.12
	Enter '.help' for instructions
	Enter SQL statements terminated with a ';'
	sqlite> .schema blog
	CREATE TABLE blog(bid INTEGER PRIMARY KEY AUTOINCREMENT,blog_user varchar,blog_server varchar,title text,content text,blog_url text, id integer, read integer, pubtime varchar);
	CREATE INDEX blog_bid ON blog(bid ASC);
	CREATE INDEX blog_id ON blog(id ASC);
	sqlite> select * from blog where bid=1;
	1|arcoolgg|csdn|海信播放 mkv |
	&#8230;
	
居然文章都在这里，那就好办了，直接用 sql 生成 wordpress 能认的 WXR 格式(其实也就是 xml)。`blog_backup` 目录里自带了很多导出格式的 sample，其中就用 wordpress 的，从中摘出一段来，从 sql 拼成 xml，试着让 wordpress 导入，却提示无效的 WXR 文件，没有版本号。看来 blog_backup 自带的格式已经不适合新版的 wordpress 了。正好手头有一个刚导出来的 WXR 文件，摘出其中的 片段，拼成了如下的语句：
	
	.output myblog.xml
	
	select ‘<?xml version=”1.0” encoding=”utf-8” ?><rss version=”2.0” xmlns:excerpt=”http://wordpress.org/export/1.0/excerpt/” xmlns:content=”http://purl.org/rss/1.0/modules/content/” xmlns:wfw=”http://wellformedweb.org/CommentAPI/” xmlns:dc=”http://purl.org/dc/elements/1.1/” xmlns:wp=”http://wordpress.org/export/1.0/”><channel><wp:wxr_version>1.0</wp:wxr_version>’ from blog where bid=1;
	
	select ‘<item><title><![CDATA['||title||']]></title><link>’||blog_url||’</link><pubDate>’||pubtime||’</pubDate><content:encoded><![CDATA['||content||']]></content:encoded><wp:post_id>’||cast ((bid + 100) as text)||’</wp:post_id><wp:post_date>’||pubtime||’</wp:post_date><wp:comment_status>open</wp:comment_status><wp:ping_status>open</wp:ping_status><wp:post_name><![CDATA['||title||']]></wp:post_name><wp:status>publish</wp:status><wp:post_parent>0</wp:post_parent><wp:menu_order>0</wp:menu_order><wp:post_type>post</wp:post_type><wp:post_password></wp:post_password><wp:is_sticky>0</wp:is_sticky><dc:creator><![CDATA[nick]]></dc:creator></item>’ from blog;
	
	select ‘</channel></rss>’ from blog where bid=1;
	
	.exit

.output 命令告诉 sqlite 将输出保存为文件
第一段 sql 用来生成 xml 的开头包括 PI 等部分。

第二段生成 节点，其中包括文章的详细信息，包括title，发布时间，文章，文章id 等。

这里有些节点很重要，比如 在上面的 sql 中，我用了 `cast((bid+100) as text)`, 是为了避免和我的 wordpress 博客中已有的文章冲突, 所以在原有的 bid 上加了 100. `wp:post_date`也很重要，决定了你发帖的时间，你肯定不希望你导入的这些帖子都是同一个发帖时间，其他的比如 title, wp:status, `wp:post_type` 等都照着写就可以了
第三段生成 xml 结尾.

.exit 让 sqlite 退出
为了方便起见，把这段脚本保持为一个文件
[sqliteinit]({{site.url}}/attachments/uploads/2011/03/sqliteinit.txt)，让 sqlite 命令行调用：

	nickx:~/Downloads $ sqlite3 -init sqliteinit.txt myblog.sqlite

执行完，sqlite 本应退出的，但不知何故 .exit 没有得到执行.

	nickx:~/Downloads $ sqlite3 -init sqliteinit.txt myblog.sqlite
	– Loading resources from sqliteinit.txt
	SQLite version 3.6.12
	Enter “.help” for instructions
	Enter SQL statements terminated with a “;”
	sqlite> .exit
	nickx:~/Downloads $ more myblog.xml
	<?xml version=’1.0′ encoding=’utf-8′ ?><rss version=’2.0′ xmlns:excerpt=’http://wordpress.org/export/1.0/excerpt/’ xmlns:content=’http://purl.org/rss/1.0/modules/content/’ xmlns:wfw=’http://wellformedweb.org/CommentAPI/’ xmlns:dc=’http://purl.org/dc/elements/1.1/’ xmlns:wp=’http://wordpress.org/export/1.0/’><channel><wp:wxr_version>1.0</wp:wxr_version><item><title><![CDATA[svn 命令行操作]]></title><link>http://blog.csdn.net/ArCoolGG/archive/2011/02/28/6214314.aspx</link><pubDate>2011-02-28 22:45</pubDate><content:encoded><![CDATA[
	<p><p>以下命令在 OSX Terminal 测试通过</p><p>
	<p>nickx:~/Documents $ svn –version</p><p>svn, version 1.6.5 (r38866)</p><p>   compiled Jun 24 2010, 17:16:45</p><div></div></p>

手动输入 .exit 退出， myblog.xml 也就生成了 (在 .output 命令中指定的）

把它导入到 wordpress，大功告成.

这里虽然是将 blog 导入到 wordpress, 但要导入到其他博客系统也是可行的，只要稍加研究一下导入格式，拼 sql 语句，生成 xml 即可
要多说一句的是，我并没有对图片进行处理，所以，新博客会外链到原博客的图片。目前还在考虑完善中~

