# nginx 安全性配置

标签（空格分隔）： nginx

---

> 主要展示在 Nginx 中配置 X-Frame-Options 、 X-XSS-Protection 、 X-Content-Type-Options 、 Strict-Transport-Security 、 https 等安全配置。

1. 不要将Nginx版本号在错误页面或服务器头部中显示

		server_tokens off;

2. 不允许页面从框架frame 或 iframe中显示，这样能避免clickjacking

		add_header X-Frame-Options SAMEORIGIN;

3. 失效某些浏览器的内容类型探嗅

		add_header X-Content-Type-Options nosniff;

4. 防止跨站脚本 Cross-site scripting (XSS) 

		add_header X-XSS-Protection "1; mode=block";

5. 激活内容安全策略Content Security Policy (CSP) 

	告诉浏览器只能从本域名和你显式指定的网址下载脚本。

		add_header Content-Security-Policy "default-src 'self'; 

		script-src 'self' 'unsafe-inline' 'unsafe-eval' 
		https://ssl.google-analytics.com 
		https://assets.zendesk.com 
		https://connect.facebook.net; 
                # 添加`data:`以支持base64方式添加图片
		img-src 'self' data:
		https://ssl.google-analytics.com 
		https://s-static.ak.facebook.com 
		https://assets.zendesk.com; 
		style-src 'self' 'unsafe-inline' 
		https://fonts.googleapis.com 
		https://assets.zendesk.com; 
		font-src 'self' 
		https://themes.googleusercontent.com; 
		frame-src 
		https://assets.zendesk.com 
		https://www.facebook.com 
		https://s-static.ak.facebook.com 
		https://tautt.zendesk.com; 
		object-src 'none'";

6. 激活会话重续提高https性能

		ssl_session_cache shared:SSL:50m;
		ssl_session_timeout 5m;

7. 激活服务器端保护免于 BEAST 攻击
	
		ssl_prefer_server_ciphers on;

8. 失效 SSLv3(自nginx 0.8.19默认激活)

		ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

9. 激活ocsp stapling

	一种机制：一个网站可以保护隐私可扩展的方式传达的证书撤销信息给访问者

		resolver 8.8.8.8;
		ssl_stapling on;
		ssl_trusted_certificate /etc/nginx/ssl/star_forgott_com.crt;

10. 配置激活HSTS(HTTP Strict Transport Security)

	避免ssl stripping

		add_header Strict-Transport-Security "max-age=31536000; includeSubdomains;";

11. redirect all http traffic to https

		server {
			listen 80;
			server_name .forgott.com;
			return 301 https://$host$request_uri;
		}




