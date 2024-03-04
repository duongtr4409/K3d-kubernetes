# Kubernetes Command

## Pod

### Pod là đơn vị nhỏ nhất về mặt phần mềm trong Kubernetes

## Secet tsl

link tham khảo: https://xuanthulab.net/su-dung-service-va-secret-tls-trong-kubernetes.html

- Tạo file xác thực trên linux

```command
openssl req -nodes -newkey rsa:2048 -keyout tls.key  -out ca.csr -subj "/CN=duowngtora.net"

openssl x509 -req -sha256 -days 365 -in ca.csr -signkey tls.key -out tls.crt
```

- Tạo kubernetes secret với các file xác thực trên

```command
kubectl create secret tls secret-nginx-cert --cert=certs/tls.crt  --key=certs/tls.key
```

- Xem ds Secret

```command
kubectl get secret -o wide
```

- Xem chi tiết Secret

```command
kubectl describe secret/[secretName]
```

- VD sử dụng Seret cho Pod trong Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
        - name: cert-volume
          secret:
            secretName: "secret-nginx-cert"
      containers:
        - name: nginx
          image: duowngtora/kubernetes:nginx
          imagePullPolicy: "Always"
          resources:
            limits:
              memory: "128Mi"
              cpu: "100m"
          ports:
            - containerPort: 80
            - containerPort: 443
          volumeMounts:
            - mountPath: "/certs"
              name: cert-volume
```
