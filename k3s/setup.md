# Set up k3s single node cluster

1. Install a linux vm, e.g. Ubuntu.
2. Execute k3s install script ([Doc](https://docs.k3s.io/quick-start))

```shell
curl -sfL https://get.k3s.io | sh -

```

3. Set up Kubeconfig

```shell
export KUBECONFIG=~/.kube/config
mkdir ~/.kube 2> /dev/null
sudo k3s kubectl config view --raw > "$KUBECONFIG"
chmod 600 "$KUBECONFIG"
```

4. Install kubernetes dashboard (optional, but recommended -> [Doc](https://docs.k3s.io/installation/kube-dashboard))

```shell
GITHUB_URL=https://github.com/kubernetes/dashboard/releases
VERSION_KUBE_DASHBOARD=$(curl -w '%{url_effective}' -I -L -s -S ${GITHUB_URL}/latest -o /dev/null | sed -e 's|.*/||')
sudo k3s kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/${VERSION_KUBE_DASHBOARD}/aio/deploy/recommended.yaml
```

5. Apply all files in [this directory](files/kub-dash)

```shell
kubectl apply -f files/kube-dash/
```

6. port forward dashboard

```shell
kubectl port-forward service/kubernetes-dashboard 8001:443
```

7. get token

```shell
kubectl -n kubernetes-dashboard create token admin-user
```

## Helm

install helm:

```shell
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## cert-manager

docs: [https://cert-manager.io/docs/installation/helm/](https://cert-manager.io/docs/installation/helm/)

```shell
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.11.0 \
  --set installCRDs=true
```

After installing the cert-manager, we have to apply the secret and cluster-issuer for let's encrpyt with cloudflare.

```shell
kubectl apply -f files/cert-amanger/
```
