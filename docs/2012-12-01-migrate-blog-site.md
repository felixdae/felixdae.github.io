---
layout: post
title: 个人博客迁移记
---

{{ page.title }}
===============

之前买的那个域名到期，又不想在那家域名商那里续期，最后只能悲催的换个新域名。其实这个新域名买到手也有一段时间了，也在vps的nginx多做过一个host配置，在原域名到期之前也可以正常访问这个博客。可是真到原域名过期之后，再用新域名进行访问，就发现页面样式全都乱掉了。

到网上搜了相关wordpress博客迁移的内容，发现要是原域名在手，迁移到新域名简直就是不费吹灰之力，只需在后台设置上做下相关修改即可。悲剧的是等我想换域名的时候，后台页面再也无法登陆上去了，总是会重定向到已经不属于我的旧域名。

作为专业技术宅当然不会就此屈服，我决定要找回我的个人博客。

第一步：查找原因，研究解决方案。

通过抓包分析发现整个页面渲染出来之后，html代码中的css路径都是旧域名之下的，同时google之后发现要解决此问题可以修改博客数据库相关内容。其实此时我本可以直接在vps上进行相关修改，但是ssh连接到这个位于美东的vps速度实在是太慢，严重影响手脑协调。另外作为“有丰富经验的大型网站业余运维人员”，未经验证直接按网帖操作也太过轻率，有负声誉，呵呵。于是决定将vps上wordpress相关程序和数据备份拷贝至本地，在本地研究试验ok再修改至远端vps。

第二步：迁移到内网。

我本地的环境是一台mbp，最初系统是os x10.7，后续升级到目前的10.8。也多亏原装的是10.7，上面预装了apache和php，无需我劳神劳力再去干没营养的装软件的活儿，听说10.8的系统就没有预装这两个软件了，好险好险。但是10.7没有mysql，好在mysql官方有mac的安装包，下载过后双击就ok。

打包备份到本地的内容包括有php的整个安装目录，nginx的安装目录，php源文件，博客的mysql数据文件。杂七杂八一起有接近上百M，美东到深圳的龟速硬是让我挂了一整晚的机才下载下来。后面想想nginx，php实在没必要整个目录打包，最多打包所有的配置文件即可，因为本地已经有了php程序的运行环境。

数据库迁移并不复杂，将博客的数据库文件夹（对应具体的那个库）拷贝到本地mysql的数据目录下即可，在本地即为/usr/local/mysql/data/。之后在进入mysql，增加新用户，进行授权，做到跟php源文件中相关配置一样即可，以保证wordpress能访问到数据。

php源文件迁移也不难，只需稍微修改源文件的访问权限即可。

apache方面要做一些修改，首先要让他能解析php文件，在本地做起来也很简单，修改/etc/apache2/httpd.conf，把#LoadModule php5_module libexec/apache2/libphp5.so反注释。并加上这样一段：

<pre>
<code>
&lt;IfModule php5_module&gt;
AddType application/x-httpd-php .php
AddType application/x-httpd-php-source .phps
&lt;IfModule dir_module&gt;
DirectoryIndex index.html index.php
&lt;/IfModule&gt;
&lt;/IfModule&gt;
</code>
</pre>

其次要为本地网站增加一个域名，我的如下：

<pre>
<code>
&lt;VirtualHost *:80&gt;
ServerAdmin webmaster@caipiaofeed.com
DocumentRoot "/usr/local/wwwroot/"
ServerName caipiaofeed.com
ServerAlias www.caipiaofeed.com
ErrorLog "/private/var/log/apache2/caipiaofeed.com-error_log"
CustomLog "/private/var/log/apache2/caipiaofeed.com-access_log" common
&lt;Directory "/usr/local/wwwroot/"&gt;
Options FollowSymLinks Includes
AllowOverride None
Order allow,deny
Allow from all
&lt;/Directory&gt;
&lt;/VirtualHost&gt;
</code>
</pre>

最后要弄一下php配置文件php.ini。

在本地环境下的php并不需要用到php.ini，但是为了博客程序能正常运行，我们需要这个文件。在我的环境下不需要从头到位再写一个php.ini，只需将/etc/php.ini.default拷贝为/etc/php.ini，然后将里面所有的关于mysql套接字的路径改为本地环境下的真实套接字路径，我的是/tmp/mysql.sock。

所有这些搞定之后，重启apache，配置相关host，应该此时已经可以访问博客内容了。

3，解决样式问题，改回外网。

虽然此时确实可以访问页面了，可是样式问题依旧。经过漫长的源码调试跟踪过程终于确定了如网帖所说，问题是在数据库上，相关内容确实要修改。此刻在本地，而且数据已经备份，改起数据库毫无压力，果断按网帖思路写了一条sql进行修改：

<pre>
<code>
update blog_options set option_value=replace(option_value, 'felixdae.com','caipiaofeed.com') where option_value like "%felixdae%";
</code>
</pre>

再次访问页面，样式终于回来了，后台也能正常登录了，原域名的阴魂终于散去。

同样的语句执行到vps数据库，也一切ok，我的博客网站终于救回来了。
