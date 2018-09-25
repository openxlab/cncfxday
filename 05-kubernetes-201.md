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
      - 192.168.100.110
EOF

kubectl logs -l component=speaker -n metallb-system

kubectl delete -f metallb.yaml

```
## Traefik Ingress controller

```yaml

kubectl create -f - <<EOF
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
- kind: ServiceAccount
  name: traefik-ingress-controller
  namespace: kube-system
EOF

curl -x http://proxy.com.cn:80 -O https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/traefik-rbac.yaml
kubectl apply -f traefik-rbac.yaml

kubectl create -f - << EOF
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
---
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: traefik-ingress-lb
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      containers:
      - image: traefik
        name: traefik-ingress-lb
        ports:
        - name: http
          containerPort: 80
        - name: admin
          containerPort: 8080
        args:
        - --api
        - --kubernetes
        - --logLevel=INFO
---
kind: Service
apiVersion: v1
metadata:
  name: traefik-ingress-service
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
    - protocol: TCP
      port: 80
      name: web
    - protocol: TCP
      port: 8080
      name: admin
  type: NodePort
EOF

kubectl create -f - << EOF
apiVersion: v1
kind: Service
metadata:
  name: traefik-web-ui
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
  - port: 80
    targetPort: 8080
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-web-ui
  namespace: kube-system
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: kube.openxlab.com
    http:
      paths:
      - backend:
          serviceName: traefik-web-ui
          servicePort: 80
EOF

kubectl --namespace=kube-system get pods

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
      - path: /wp
        backend:
          serviceName: wordpress
          servicePort: 80
EOF

```
