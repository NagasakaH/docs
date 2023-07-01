# Kubernetesを導入する

参考:  
[https://gaganmanku96.medium.com/kubernetes-setup-with-minikube-on-wsl2-2023-a58aea81e6a3](https://gaganmanku96.medium.com/kubernetes-setup-with-minikube-on-wsl2-2023-a58aea81e6a3)  
[https://joepreludian.medium.com/how-to-start-up-minikube-automatically-via-system-d-2cad99fd79bf](https://joepreludian.medium.com/how-to-start-up-minikube-automatically-via-system-d-2cad99fd79bf)

## minikubeのインストール

```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x ./minikube
sudo mv ./minikube /usr/local/bin/
minikube config set driver docker
```

## kubectlのインストール

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/
```

## systemdでminikubeを起動するようにする

```title="/etc/systemd/system/minikube.service"
[Unit]
Description=Kickoff Minikube Cluster
After=docker.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/minikube start
RemainAfterExit=true
ExecStop=/usr/local/bin/minikube stop
StandardOutput=journal
User=<ユーザー名>
Group=<ユーザー名>

[Install]
WantedBy=multi-user.target
```
