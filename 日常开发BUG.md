# ./nginx -s reload 报nginx: [error] invalid PID number "" in "/run/nginx.pid"

需要重新加载配置文件

```bash
$ nginx -c /etc/nginx/nginx.conf
```

如执行过程中显示

```bash
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:8080 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:8080 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:8080 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:8080 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:8080 failed (98: Address already in use)
nginx: [emerg] still could not bind()
```

表示nginx在已经在进程中执行m需要把执行中nginx的进程杀掉,无论整样stop或restart都没作用,显示占用80端口的进程

```bash
$ fuser -n tcp 80
80/tcp:              27814 29621 29622 29623 29624
$  kill -9 27814 29621 29622 29623 29624
```

再重新指定加载文件,重启nginx

```bash
$ nginx -c /etc/nginx/nginx.conf
$ nginx -s reload
```

