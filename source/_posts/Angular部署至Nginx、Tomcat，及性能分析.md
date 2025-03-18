---
title: Angular部署至Nginx、Tomcat，及性能分析
date: 2019-01-18 16:18:40
tags:
---

## Angular部署至Nginx、Tomcat，及性能分析
#### 打包Angular工程

```
ng build --prod
```
工程目录下的dist是构建后的代码
#### 部署至Nginx
将dist中的代码复制到一个指定目录，例如

linux：/usr/local/ngweb

windows: D:/ngweb/

> 修改nginx/conf/nginx.conf

```
server{
	listen 80;
	server_name  localhost;
	location / {
	    root   /usr/local/ngweb/;
	    index  index.html;
    	try_files $uri $uri/ /index.html;
    }
}
```
try_files $uri $uri/ /index.html; 若找不到匹配路径则匹配/index.html

启动nginx
```
    /usr/local/nginx/sbin/nginx
```
重启nginx
```
    /usr/local/nginx/sbin/nginx -s reload
```

#### 部署至Tomcat
修改app.routing.module.ts，使用useHash，如果不设置的话，在路由地址发生变化后刷新页面会导致404错误
```
    @NgModule({
        imports: [RouterModule.forRoot(routes, { useHash: true })],
        exports: [RouterModule]
    })
```
将dist中的代码复制到 tomcat/webapps/ROOT/ 下,默认端口为8080

若需要将端口修改为80则在 tomcat/conf/server.xml 中设置

```
<Connector port="80" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
```
启动tomcat

```
    sh /usr/local/tomcat/bin/startup.sh
```
停止tomcat

```
    sh /usr/local/tomcat/bin/shutdown.sh
```

#### 性能对比
- 环境：1核2G内存Centos7 (VMWare)、[Nginx(1.15.8)](http://nginx.org/en/download.html)、[Tomcat(7.0.65)](https://archive.apache.org/dist/tomcat/tomcat-7/v7.0.65/bin/)
- 工具： [Apache-JMeter](https://jmeter.apache.org/)

 **100并发**
>Nginx
![Nginx100并发](https://i-blog.csdnimg.cn/blog_migrate/51d259ff52b0cd32f317099492b85fef.png)
Tomcat
![Tomcat100并发](https://i-blog.csdnimg.cn/blog_migrate/c398358c0e1cea1445e79100f352e9d1.png)

 **200并发**
>Nginx
![Nginx200并发](https://i-blog.csdnimg.cn/blog_migrate/ac881c74506d591e65482d15559b27fd.png)
Tomcat
![Tomcat200并发](https://i-blog.csdnimg.cn/blog_migrate/a558699d8826bcbc6ba0cd5e3aecb0b5.png)

 **500并发**
>Nginx
![Nginx500并发](https://i-blog.csdnimg.cn/blog_migrate/726a86e7317eed8aae2359fb551fcc4d.png)
Tomcat
![Tomcat500并发](https://i-blog.csdnimg.cn/blog_migrate/cb0b52441ddc0739d8ad0f1910543112.png)

通过对比，Nginx在高并发下性能较Tomcat更好