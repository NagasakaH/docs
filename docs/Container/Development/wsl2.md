# WSL2の環境構築

## WSL2の導入

## WSL2を有効にする

```title="powershell"
wsl --set-default-version 2
```

## Ubuntuの導入

```title="powershell"
wsl --install -d Ubuntu-22.04
wsl --set-default Ubuntu-22.04
```

## systemdの有効化

最新版にする

```title="powershell"
wsl --update
```

systemdを有効にする

```title="/etc/wsl2.conf"
[boot]
systemd=true
```

システムを一度終了する

```title="powershell"
wsl --shutdown
```
