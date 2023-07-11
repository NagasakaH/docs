## 同一ネットワークの任意のコンテナの任意のポートにサブドメインを使って転送する仕組み

### サブドメインを正規表現で分離してポートとホストを取得、転送するnginxの設定

```bash
docker network create share
```

```yml title="compose.yml"
services:
  nginx:
    image: nginx:latest
    ports:
      - 80:80
    volumes:
      - ./default.conf:/etc/nginx/conf.d/default.conf
    networks:
      - share

networks:
  share:
    external: true
```

```conf title="default.conf"
server {
    listen 80;
    server_name ~^(?<port>[0-9]*)-(?<container>.*).localhost;

    location / {
      resolver 127.0.0.11;
      proxy_pass http://$container:$port;
      proxy_pass http://$container:$port;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection upgrade;
      proxy_set_header Accept-Encoding gzip;
    }
}
```

### ワイルドカードドメインを有効にするために独自でDNSを建てる

```yml title="compose.yml"
version: '3.1'
services:
  coredns:
    image: coredns/coredns:1.7.0
    container_name: coredns
    restart: on-failure
    expose:
      - '53'
      - '53/udp'
    command: -conf /etc/coredns/Corefile
    ports:
      - '53:53'
      - '53:53/udp'
    volumes:
      - './config:/etc/coredns'
```

```conf title="config/Corefile"
{
    whoami
    forward . 8.8.8.8:53
    errors
    log . "{proto} {remote} is Request: {name} {type} {>id}"

    # 自前のレコード（ワイルドカード）
    template IN A mypc {
        match "^([^\.]+)\.mypc\.$"
        answer "{{ index .Match 1 }}.mypc. IN A 127.0.0.1"
        fallthrough
    }
    reload
}
```

### 転送先のサービスを立ち上げる

```yml title="comopse.yml"
services:
  dest:
    image: nginx:latest
    networks:
      - share
netrowks:
  share:
    external: true
```

正常に転送されることを確認する

```
curl 80-dest.mypc
```
