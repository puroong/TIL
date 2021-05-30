# Root vs Alias

/static로 시작하는 경로에 대해서만 nginx로 정적 파일을 서빙하고자 했음

그 과정에서 알게된 root와 alias의 차이점에 대해 작성하려함


```
server {
    listen 443;

    location /static {
        root /home/ubuntu/zoomkr-static;
    }

    ...
}
```

![](/assets/nginx-root.png)

**root는 prefix도 파일 경로에 포함함**

```
server {
    listen 443;

    alias /home/ubuntu/zoomkr-static;

    ...
}
```

![](/assets/nginx-alias.png)

**alias는 prefix를 파일 경로에서 제외함**
