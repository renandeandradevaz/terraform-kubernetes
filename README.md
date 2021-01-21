# terraform-kubernetes (AWS)

https://renandeandradevaz.medium.com/using-terraform-to-create-a-kubernetes-cluster-and-a-nginx-ingress-242fac453a7c

## (PART 1)

This repo has some resources to start using Kubernetes and terraform.
When you finish this tutorial, you will have this resources created on your AWS account:

* EKS cluster
* VPC, Subnets, Nat gateway, etc
* IAM policy
* 2 kubernetes pods for tests
* 1 AWS Load balancer
* 1 ingress-nginx to receive requests and proxy for pods


First, we need to clone the repo and init terraform:
```
git clone git@github.com:renandeandradevaz/terraform-kubernetes
cd terraform-kubernetes
terraform init
```

Then, we have to apply the resources on the cloud:
```
terraform apply
```

Create an environment variable to keep kubeconfig data:
```
export KUBECONFIG=${PWD}/kubeconfig_my-cluster
```

Create the nginx ingress inside the cluster:
```
kubectl apply -f ingress-nginx.yaml
```

Deploy two containers for testing:
```
kubectl apply -f banana.yaml
```
```
kubectl apply -f apple.yaml
```

Deploy the routes:
```
kubectl apply -f routes.yaml
```

If you want to activate the auto scaler. You must run:
```
kubectl apply -f auto-scaler.yaml
```
PS: The cluster's name is inside of the file above. The name is "my-cluster". If you choose a different name for the cluster, you must rename the cluster inside the file.


---


After create all the resources, now it's time to test.
Go to https://console.aws.amazon.com/ec2/v2/home#LoadBalancers and copy the DNS name


Test if application returns something:
```
curl {AWS_LOAD_BALANCER_URL}/banana
``` 
and 
```
curl {AWS_LOAD_BALANCER_URL}/apple
``` 

In the end, you can remove all created resources to save money:

```
terraform destroy
```


---

# Kubernetes logs going to Kibana and s3

LINK_AQUI

## (PART 2)

In this section, we are going to setup an ELK environment inside the kubernetes.
The logstash plugin will send the logs to kibana and s3.

After the cluster created, you have to execute the following commands:


First, we need to install the elasticsearch in our cluster. The command is:
```
kubectl apply -f elasticsearch-ss.yaml
```


Then, we are going to install the logstash. Logstash will be responsible to send the container logs to elasticsearch and s3.
In this file, we have the name of kubernetes indexes and the buckets and directories on s3. 
Before execute the command, you must change the bucket name and region name inside the file.
Then, execute:
```
kubectl apply -f logstash-deployment.yaml
```


Now, install filebeat. Filebeat is the plugin responsible to obtain the container logs. Logstash connects with filebeat.
```
kubectl apply -f filebeat-ds.yaml
```

(Optional) If you also want to have the kubernetes metrics in kibana, you can install the metric beat plugin.
```
kubectl apply -f metricbeat-ds.yaml 
```

Install kibana:
```
kubectl apply -f kibana-deployment.yaml
```


Install a cron job to delete old logs in elasticsearch (You can configure the period inside this file. The current is 30 days)
```
kubectl apply -f curator-cronjob.yaml
```

And, in the end, we are going to install a simple application to verify if all was configured successfully
```
kubectl apply -f json-log-generator.yaml
```
Notice that if you run the command "kubectl logs json-log-generator-app -n kube-system", you will see that the application is using the JSON format to write the logs. And it has a field named "app". This field is very important. Because the logstash uses this field to know which elasticsearch index will be used and s3 directory, as well.
For example, if you install your application inside this configured cluster, use the json format to log the data, and the value of app field in the json is "my-app" or "my-awesome-app", this index will be created in elasticsearch and a folder will that name will be created on s3.


To verify if the logs are going to s3, you can verify directly on your bucket (The rotation period configured is 10 minutes)

To verify if the logs are going to kibana, you have to do that:


Get the name of the kibana pod:
```
kubectl get pods -n kube-system
```

And open a tunnel to access kibana in port 5601
```
kubectl port-forward -n kube-system {KIBANA_LOGGING_POD_NAME_HERE} 5601:5601
```

And then, access the URL on your machine "localhost:5601"
Configure the name of the index on kibana as "my-app*"
And you will see the logs there.

At the end, remember to remove the resources to dont increase your bill at aws:
```
terraform destroy
```
