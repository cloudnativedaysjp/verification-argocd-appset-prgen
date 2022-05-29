# verification-argocd-appset-prgen

Argo CD ApplicationSet の PullRequest Generator 機能の検証用リポジトリ

## 検証環境の構築

* Argo CD 用の secret を用意

```bash
read -s ARGOCD_ADMIN_PASSWORD
kubectl create secret generic argocd-secret -oyaml --dry-run=client \
  --from-literal=admin.password=$ARGOCD_ADMIN_PASSWORD | kubectl neat \
  > argocd/overlays/dev-with-secrets/secret-argocd.yaml
```

```bash
read -s GITHUB_TOKEN
kubectl create secret generic github-token -oyaml --dry-run=client \
  --from-literal=token=$GITHUB_TOKEN  | kubectl neat \
  > argocd/overlays/dev-with-secrets/secret-github-token.yaml
```

* Argo CD のインストール

```bash
kustomize build argocd/overlays/dev-with-secrets | kubectl apply -f-
```

## 検証

* Review Apps 環境構築用マニフェスト (ApplicationSet) の適用
    * 対象 app-repo: https://github.com/cloudnativedaysjp/dreamkast
    * 対象 labels: `reviewapps/ignore`

```bash
kubectl apply -f reviewapps/dreamkast-dk.yaml
```

* Application が生えていることの確認
    * HEALTH STATUS が Progressing のままであることは意図通り (検証環境には Ingress controller が存在しないため)

```bash
$ kubectl get app -n argocd
NAME                SYNC STATUS   HEALTH STATUS
dreamkast-dk-1207   Synced        Progressing
dreamkast-dk-1235   Synced        Progressing
dreamkast-dk-1237   Synced        Progressing
```

* Review Apps 環境に Deployment, Service, Ingress が存在することの確認

```
$ kubectl get deployment,svc,ing -n dreamkast-dk-1207
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   3/3     3            3           10m

NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/nginx   ClusterIP   10.96.122.221   <none>        80/TCP    10m

NAME                              CLASS    HOSTS                                  ADDRESS   PORTS   AGE
ingress.networking.k8s.io/nginx   <none>   dreamkast-dk-1207.cloudnativedays.jp             80      10m
```
