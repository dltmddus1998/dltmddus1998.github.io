# Nginx를 통해 웹의 User Agent가 모바일일 때 도메인 변경하기

## ⁉ How to...
> 현재 nginx 설정은 다음과 같다. (각각 웹 버전과 모바일 버전 따로 있다.)
> 

```
server {
	listen 443 ssl http2;
	listen [::]:443 ssl http2;

	server_name exmple.com;
	
	root /var/www/example/public;
	...
	...
}
```

```
server {
	listen 443 ssl http2;
	listen [::]:443 ssl http2;

	server_name m.exmple.com;
	
	root /var/www/example-mobile/public;
	...
	...
}
```

> example.com에서 m.example.com으로 리다이렉트하도록 설정을 추가해준다.
> 

[***example.com](http://example.com) → [m.example.com](http://m.example.com)  (redirect) 추가***

```
server {
	listen 443 ssl http2;
	listen [::]:443 ssl http2;

	server_name exmple.com;
	
	root /var/www/example/public;

	set $mobile_rewrite do_not_perform;
	
	if ($http_user_agent ~* "(android|bb\d+|meego.....)) {
       set $mobile_rewrite perform;
   } 
   if ($mobile_rewrite = perform) { **모바일일 경우**   
      rewrite ^ https://m.$server_name$request_uri redirect;
      break;
   }
	if ($mobile_rewrite = do_not_perform) { **웹일 경우**
		rewrite ^ https://example.com$request_uri redirect;
		break;
	}
   ...
   ...
}
```

✓ 위 코드는 가변적인 `$mobile_rewrite` 값을 *do_not_perform* 값에 설정한다.

✓ if문을 통해 user agent가 mobile agent로 구성되어 있는지 체크한다.

✓ mobile agent라면, `$mobile_rewrite` 값을 *perform*으로 변경한다.

Nginx는 가변적인 변화들에 대해 알린다.

✓ Nginx가 perform 값을 감지하면 (모바일 상태), 현재 url을 모바일 url로 변경시킨다.

> 테스트를 해보면 다음과 같다.
> 

```
> sudo nginx -t

> sudo service nginx restart
```

## Reference

[Detect Mobile User Agent in Nginx](https://medium.com/geekculture/detect-mobile-user-agent-in-nginx-806a43f5782a)

## 실제 .conf 설정 코드

```
server {
  listen 80;

  location / {
          proxy_pass http://127.0.0.1:8000;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
  }
}

server {
  listen 80;
  server_name [site name];


  set $mobile_rewrite do_not_perform;

   if ($http_user_agent ~* "(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge |maemo|midp|mmp|mobile.+firefox|netfront|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows ce|xda|xiino") {
          set $mobile_rewrite perform;
  }

  if ($mobile_rewrite = perform) { 디스플레이 크기가 모바일일 경우
          rewrite ^ https://m.$server_name$request_uri redirect;
          break;
  }

  if ($mobile_rewrite = do_not_perform) { 디스플레이 크기가 웹일 경우
          rewrite ^ https://$server_name$request_uri redirect;
          break;
  }
}
```
