# kubectl

自分用リファレンス  
[公式のチートシート](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

### Pod(コンテナ)の実行

※1Podで複数コンテナも動かせる  
※1PodにつきIPが一つ付与される

[ドキュメント](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run)  
コンテナの実行  
--envは同じような感じで使える

```bash
kubectl run <pod名> --image=nginx:alpine --env VARIABLE=1
```

"docker run --rm <イメージ名> -d" 相当  

```bash
kubectl run <pod名> --image=<イメージ名> --restart=Never
```

任意のコマンドを実行させたい場合は--commandを使う
```bash
kubectl run <pod名> --image=<イメージ名> --restart=Never --command -- <コマンド名> <引数1> <引数2>
```

外部にポートを公開したい場合はkubectl exposeと組み合わせる必要がある
```bash
kubectl run <pod名> --image=<イメージ名> --restart=Never --port <公開したいコンテナのポート番号>
```


### 実行しているPodを確認する

[ドキュメント](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get)

```bash
kubectl get pods
# -o wideを付けると詳細情報が表示される
kubectl get pods -o wide
# 全てのネームスペースのPodを表示
kubectl get pods --all-namespaced
# podの詳細をyamlで出力
kubectl get pod <Pod名> -o yaml
```

### 実行しているPodを削除する

[ドキュメント](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete)

```bash
# Pod名を指定して削除
kubectl delete pod <pod名>
# Pod作成に使用したファイルを指定して削除
kubectl delete -f <作成に使用したファイル>
```
