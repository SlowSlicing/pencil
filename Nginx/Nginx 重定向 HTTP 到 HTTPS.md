```
server {

	......

	rewrite ^(.*)$ https://$host$1 permanent; 【这是 ngixn 早前的写法，现在还可以使用】

	location / {
		......
	}

}
```

```
server {

	......

	return 301 https://$server_name$request_uri; 【这是 nginx 最新支持的写法】

	location / {
		......
	}

}
```
