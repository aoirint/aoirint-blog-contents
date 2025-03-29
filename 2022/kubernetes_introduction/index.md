# minikubeによるKubernetes入門

- minikube: <https://github.com/kubernetes/minikube>
- <https://kubernetes.io/ja/docs/setup/learning-environment/minikube/>
- <https://qiita.com/progrhyme/items/116948c9fef37f3e995b>


## minikubeのインストール

- <https://minikube.sigs.k8s.io/docs/start/>

```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb
```

## kubectlのインストール

- <https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/>

```shell
sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

## minikubeの開始

```shell
minikube start
```

## Podの一覧を確認

- <https://kubernetes.io/ja/docs/reference/kubectl/cheatsheet/#%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9%E3%81%AE%E6%A4%9C%E7%B4%A2%E3%81%A8%E9%96%B2%E8%A6%A7>

```shell
kubectl get po -A
# Or
kubectl get pods --all-namespaces
```
