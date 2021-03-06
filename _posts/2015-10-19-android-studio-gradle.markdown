---
layout: post
title:  "Android Studio Gradle 导出JavaDocJar提示编码GBK的字符无法映射解决办法"
date:   2015-10-19 11:02:15
categories: Android开发
tags: AndroidStudio Gradle 编码
---
最近因为要把PreIOC放入jcenter中，百度找了很多方法，终于有点眉头了，但是却卡在了生成JavaDocJar，因为要放入jcenter中必须要上传生成的jar、sourcejar、和JavaDocJar，刚开始百度了很久，都是说下面2种方法
##### 第一种：
修改项目和IDE的编码格式：file->setting-file Encodings
##### 第二种：
修改build.gradle,添加如下代码
{% highlight c++ linenos%}
tasks.withType(JavaCompile) {
    options.encoding = "UTF-8"
    options.compilerArgs << "-Xlint:unchecked"
}
{% endhighlight %}
但是上面2种只能是编译的时候不出问题，到处JavaDocJar的时候还是一样报错（当然也有可能是我导出的方法有问题），后面没有办法，只能吧文件全部改为ASCII编码的，成功编译了。

后来也就是今天，因为PreIOC要更新新版本，由于文件修改为了ASCII编码的，中文全部乱码了，最后还是决定找一个解决办法，于是各种百度谷歌，最后，终于通过谷歌找到了。
##### 配置文件如下
{% highlight c++ linenos%}
javadoc {
    options{
        encoding "UTF-8"
        charSet 'UTF-8'
        author true
        version true
        links "http://docs.oracle.com/javase/7/docs/api"
        title PROJ_ARTIFACTID
    }
}
{% endhighlight %}
配置之后就可以成功导出JavaDocJar了
