# Despliegue en AWS

### Herramientas a instalar

- Chocolatey (Windows)
- Brew (MAC)
- aws-cli (https://awscli.amazonaws.com/AWSCLIV2.msi)
- eksctl (choco install eksctl -y)
- helm (choco install kubernetes-helm -y)

### Configurar usuario

```
aws configure
```

### Crear cluster

```
eksctl create cluster --name cluster-docker --without-nodegroup --region us-east-2 --zones us-east-2a,us-east-2b
```

### Agregar nodos

```
eksctl create nodegroup --cluster cluster-docker --name cluster-docker-nodegroup  --node-type t3.medium --nodes 1 --nodes-min 1 --nodes-max 3 --asg-access --region us-east-2
```

### Crear IAM Provider

```
eksctl utils associate-iam-oidc-provider --cluster cluster-docker --approve --region us-east-2
```

### Descargar políticas para el cluster

```
curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.1.2/docs/install/iam_policy.json
```

### Crear la política usando un archivo json

```
aws iam create-policy --policy-name AWSLoadBalancerControllerPolicy --policy-document file://iam_policy.json
```

### Crear cuenta ServiceAccount para el cluster

```
eksctl create iamserviceaccount --cluster=cluster-docker --namespace=kube-system --name=aws-load-balancer-controller --attach-policy-arn=arn:aws:iam::282865065290:policy/AWSLoadBalancerControllerPolicy --override-existing-serviceaccounts --approve --region us-east-2
```

### Verificar si existe el ingress controller del balanceador

```
kubectl get deploy -n kube-system alb-ingress-controller
```
