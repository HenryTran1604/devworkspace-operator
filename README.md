# Helm chart cho DevWorkspace Operator
## 1. Thực hiện
### a. Tiền điều kiện
#### Cài Ingress
- Hiện em dùng minikube nên chỉ cần: `minikube addons enable ingress`
#### Cài cert-manager
```sh
helm repo add jetstack https://charts.jetstack.io --force-update
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.16.2 \
  --set crds.enabled=true
```
### b. Tạo helm chart
- Tạo helm chart
```
helm create devworkspace
```
- Lấy các file YAML từ repository của devworkspace-operator [tại đây](https://github.com/devfile/devworkspace-operator/tree/main/deploy/deployment/kubernetes)
> Trong đó, [`combined.yaml`](https://github.com/devfile/devworkspace-operator/blob/main/deploy/deployment/kubernetes/combined.yaml) chứa tất cả các định nghĩa crds, deployments, và service. Và [`default-config.yaml`](https://github.com/devfile/devworkspace-operator/blob/main/deploy/default-config.yaml) chứa thông tin cấu hình.
- file `values.yaml`
```yml
namespace: devworkspace-controller
routingSuffix: 192.168.49.2.nip.io
defaultRouting: basic
pullPolicy: IfNotPresent
```
- Cài helm chart.
```
helm install devworkspace-operator .
```
## 2. kết quả
- Trạng thái cụm sau khi cài

![after-setup](./images/after-setup.png)
> ImagePullBackOff vì proxy, em dùng 4G thì nó pull về và chạy bình thường.
- Sử dụng 1 YAML file để định nghĩa cho project được clone
```YAML
kind: DevWorkspace
apiVersion: workspace.devfile.io/v1alpha2
metadata:
  name: go-cde
spec:
  started: true
  template:
    projects:
      - name: outyet
        git:
          remotes:
            origin: https://github.com/l0rd/outyet
    components:
      - name: devtools
        container:
          image: quay.io/devfile/universal-developer-image:ubi8-latest
          memoryRequest: 2G
          memoryLimit: 8G
          cpuRequest: '1'
          cpuLimit: '4'
  contributions:
    - name: vscode
      uri: https://eclipse-che.github.io/che-plugin-registry/main/v3/plugins/che-incubator/che-code/latest/devfile.yaml
      components:
        - name: che-code-runtime-description
          container:
            env:
              - name: CODE_HOST
                value: 0.0.0.0
```
- Apply file này vào cụm
```sh
kubectl apply -f dw.yaml
```
- Kết quả:
![cde](./images/cde.png)
![img](./images/image.png)

# II. Cập nhật lại cấu trúc thư mục
- Các custom-resource-definition được lưu trong [`crds`](./crds/)
- Tách các RBAC và service vào các folder tương ứng
  - edit-workspaces
  - leader-election
  - metrics
  - proxy
  - view-workspaces