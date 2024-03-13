# Kubernetes \_ Ubuntu (Devops mentor):

link: https://www.youtube.com/watch?v=Zq20HguI_SA&list=PLVx1qovxj-akr_3XqQQgpqRyQw4GYuS4h&index=2

## Cài đặt docker

- tùy từng Os System cài đặt docker
  VD: với Ubuntu (có thể tham khảo cài đặt theo hướng dẫn ở link này: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04)

```command
sudo apt update

sudo apt install apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"

apt-cache policy docker-ce

sudo apt install docker-ce

sudo systemctl status docker

sudo usermod -aG docker ${USER}

su - ${USER}

groups

sudo usermod -aG docker {{username}}
```

## Cài đặt kubectl, kubeadm, kubelet

link: có thể cài đặt theo hướng dẫn ở link này: https://www.linuxtechi.com/install-kubernetes-on-ubuntu-22-04/

- Add Kubernetes Signing Key

```command
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

- Add Software Repositories

```command
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

```command
sudo apt update
```

- Install Kubernetes Tools

1. install command:

```command
sudo apt install -y kubelet kubeadm kubectl
```

2. Mark the packages as held back to prevent automatic installation, upgrade, or removal:

```command
sudo apt-mark hold kubelet kubeadm kubectl
```

ps: 
- nếu gặp vấn đề với kubelet chạy lệnh sau (kubelet không start được)

```
sudo swapoff -a

sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
- nếu lỗi 403 Forbidden ở bước 'Add Kubernetes Signing Key' thì cài đặt warp-cli
```command
# Add cloudflare gpg key
curl -fsSL https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --yes --dearmor --output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg

# Add this repo to your apt repositories
echo "deb [signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list

# Install
sudo apt-get update && sudo apt-get install cloudflare-warp
```

## Cài đặt và sử dụng k3d tạo kubernetes cluster

- cài đặt k3d

```command
wget -q -O - https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
```

- sử dụng k3d khởi tạo kubernetes cluster

##### Khởi cluster đơn node

```command
k3d cluster create {{ClusterName}}
```

##### Khởi tạo cluster đa node

```command
k3d cluster create {{ClusterName}} --agents 2 --servers 1
```

ps:
<br> --agents 2: 2 là số lượng node worker
<br> --servers 1: 1 là số lượng node master

- sử dụng k3d để tạo node và add vào kubernetes cluster

```command
k3d node create {{NodeName}} --role agent --cluster {{ClusterName}}
```

ps:
<br> {{NodeName}}: NodeName là tên cluster muốn thêm node vào
<br> --cluster {{ClusterName}}: ClusterName là tên cluster muốn thêm node vào
<br> --role {{agent}}: agent là role của node trong cluster muốn thêm node vào (agent: node workder | server: node master)

## Một số lệnh thông dụng trong k3d

- Lấy thông tin danh sách cluster

```command
k3d cluster list
```

- Stop cluster

```command
k3d cluster stop {{ClusterName}}
```

- Start cluster

```command
k3d cluster start {{ClusterName}}
```

- Delete cluster

```command
k3d cluster delete {{ClusterName}}
```

- Lấy thông tin danh sách node

```
k3d node list
```

## Chuyển môi trường làm việc với kubernetes cluster (thao tác với các cluster khác nhau)

- Xem thông tin cluster hiện tại

```command
kubeclt cluster-info
```

- Xem thông tin danh sách các cluster có thể kết nối

```command
kubectl config get-contexts
```

- Chuyển đổi cluster để làm việc

```command
kubectl config ues-context {{ClusterName}}
```

#### Thêm thông tin cluster mới vào trong kubernetes context để có thể chuyển đổi làm việc

- b1: lấy thông tin cấu hình của cluster cần kết nối đến (muốn thao tác dòng lệnh)

```command
    ### vd: dùng scp để copy file cấu hình về máy ###
    scp root@192.168.10.100:/etc/kubernetes/admin.conf ~/.kube/config-mycluster
```

ps: Nếu muốn yêu cầu kubectl sử dụng ngay file cấu hình nào đó, thì gán biến môi trường KUBECONFIG bằng đường dẫn file cấu hình, ví dụ sử dụng file cấu hình config-mycluster
(Sau lệnh đó thì kubectl sẽ dùng config-mycluster để có thông tin kết nối đến, nhưng trường hợp này chỉ có hiệu lực trong một phiên làm việc, ví dụ nếu bạn đóng terminal và mở lại thì lại phải thiết lập lại biến môi trường như trên.)

```command
    export KUBECONFIG=/Users/DuowngTora/.kube/config-mycluster
```

- b2: Thực hiện kết hợp 2 file: config và config-mycluster thành 1 và lưu trở lại config.

```command
    export KUBECONFIG=~/.kube/config:~/.kube/config-mycluster
    kubectl config view --flatten > ~/.kube/config_temp
    mv ~/.kube/config_temp ~/.kube/config
```
