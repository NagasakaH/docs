# 小ネタ集

## DBの起動待ち

[initContainers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)を使うのが一般的？  
[wait-for](https://www.patrickdap.com/post/wait-for/)このwait-forコンテナを使うのもアイデアとしては良さそう

Postgresの場合はpg_isreadyで待つのが恐らく無難  
※ DBとバックエンドのPODは別にする必要あり  

```yaml title="database-deployment.yaml"
apiVersion: v1
kind: Pod
metadata:
  name: wait-db-demo
spec:
  containers:
  - name: database
    image: postgres:10
```

```yaml title="backend-deployment.yaml"
apiVersion: v1
kind: Pod
metadata:
  name: wait-db-demo
spec:
  initContainers:
  - name: check-db-ready
    image: postgres:9.6.5
    command: ['sh', '-c', 
      'until pg_isready -h postgres -p 5432; 
      do echo waiting for database; sleep 2; done;']
  containers:
  - name: backend-service
    image: <バックエンドイメージ>
```
