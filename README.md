1. 於minikube安裝Argo CD
```
kubectl create namespace argocd
kubectl apply  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml -n argocd

kubectl port-forward svc/argocd-server -n argocd 8080:443
```
2. 取得Argo CD帳號密碼[參考文章](https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli)，預設帳號為`admin`，可用argo CLI下`argocd admin initial-password`，或是用K8S指令取得，並decode base64。
```
#
yangtui@YangtuiPC:~/SRE-Program-Batch1$ kubectl get secrets argocd-initial-admin-secret -n argocd -o yaml
apiVersion: v1
data:
  password: aWJ6MzJaR0dqVDJDWEdhMA==
kind: Secret
metadata:
  creationTimestamp: "2024-04-20T09:43:21Z"
  name: argocd-initial-admin-secret
  namespace: default
  resourceVersion: "4992"
  uid: 54d6f178-f8a3-40aa-9270-90a0f59c3f35
type: Opaque

#
echo eG9ETmJJTG85bUEyWDhGOA== | base64 --decode 
xoDNbILo9mA2X8F8
```

3. 可以用UI建立`new app`，這邊用k8s yaml建立
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-argo-application
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/timmy304681/argocd-app-config.git
    targetRevision: main
    path: dev
  destination: 
    server: https://kubernetes.default.svc
    namespace: myapp

  syncPolicy:
    syncOptions:
    - CreateNamespace=true

    automated:
      selfHeal: true
      prune: true
```
![alt text](images/image.png)


# Github action workflow

k8s repository :https://github.com/timmy304681/argocd-app-config


1. push和pull_request皆會做k8s-lint和yaml lint檢查 ，CI URL: https://github.com/timmy304681/argocd-app-config/actions/runs/8769916068/job/24065909777
2. 若發出pull_request會觸發yaml lint檢查，若未通過，action會失敗且發comment通知失敗錯誤處，開發者可依內容修正，重新push
![alt text](images/image-2.png)
成功會印出如下，review人員就可merge
![alt text](images/image-3.png)
3. 為了實作方便設計兩個環境`dev` & `prod`，`dev`環境會偵測repository內`dev`folder，`prod`則偵測`prod`folder，
4. `prod`會關閉automated application synchronization，以確保repository的prod內資料異動，不會隨意自動被部屬
![alt text](images/image-4.png)
![alt text](images/image-7.png)
![alt text](images/image-8.png)
![alt text](images/image-9.png)

