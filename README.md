# terraform-kubernetes


```
terraform init
```

```
terraform apply
```

```
export KUBECONFIG=${PWD}/kubeconfig_my-cluster
```

```
kubectl apply -f ingress-nginx.yaml
```

```
kubectl apply -f banana.yaml
```

```
kubectl apply -f apple.yaml
```

```
kubectl apply -f routes.yaml
```

Go to https://console.aws.amazon.com/ec2/v2/home#LoadBalancers and copy the DNS name


Test if application returns something

```
curl {AWS_LOAD_BALANCER_URL}/banana
``` 

and 

```
curl {AWS_LOAD_BALANCER_URL}/apple
``` 