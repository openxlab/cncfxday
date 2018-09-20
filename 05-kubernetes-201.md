# 05 Kubernetes 201

## MetalLB

Install MetalLB

```yaml
curl -x http://proxy.com.cn:80 -O https://raw.githubusercontent.com/google/metallb/master/manifests/metallb.yaml
kubectl apply -f metallb.yaml
kubectl get pods -n metallb-system

kubectl apply -f - << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: openxlab-ip-space
      protocol: layer2
      addresses:
      - 192.168.100.111
EOF

kubectl logs -l component=speaker -n metallb-system

kubectl delete -f metallb.yaml

```

## NGINX Ingress Controller

```yaml
curl -x http://proxy.com.cn:80 -O https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
kubectl apply -f mandatory.yaml
curl -x http://proxyhk.zte.com.cn:80 -O https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml
kubectl apply -f service-nodeport.yaml

kubectl get pods --all-namespaces -l app.kubernetes.io/name=ingress-nginx --watch

kubectl create -f - << EOF
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: wp-ingress
spec:
  rules:
  - host: wordpress.openxlab.com
    http:
      paths:
      - path: /
        backend:
          serviceName: wordpress
          servicePort: 80
EOF

```
