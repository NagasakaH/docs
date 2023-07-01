# Dockerを導入する

参考: [公式の導入手順](https://docs.docker.com/engine/install/ubuntu/)  
↑の手順通りにインストールした上で[default-address-poolsの変更](#default-address-poolの変更)を適応すれば良いです

## Dockerインストール
### 不要なパッケージの削除

```title="bash"
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

### 必要なパッケージのインストール

```title="bash"
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
```

### GPG Keyの追加

```title="bash"
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

### リポジトリの追加

```title="bash"
echo \
 "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
 "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
 sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Docker Engineのインストール

```title="bash"
sudo apt update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin 
```

## 設定変更

### default-address-poolの変更

Dockerの使うIPのレンジとwindows側のアドレスが衝突して名前解決ができなくなるのを回避する

```title="/etc/docker/daemon.json"
{
  "default-address-pools":
    [
      {"base":"172.16.0.0/12", "size":24}
    ],
  "dns": ["8.8.8.8"]
}
```

設定を反映する

```bash
systemctl daemon-reload
systemctl restart docker
```
