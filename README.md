
# Hello World Application.

## EKS Clutser Provision

I have **aws-eks** folder where I written all the terraform code to provision it in any region, to create eks cluster just need to run below terraform commands.
```
terraform init
```
### terraform init will install all the plugins for us and prepare our code to provision.

```
terraform plan
```
### terraform plan will execute our code as a dry run and will show us changes that are going to happen as Create, Update or Destroy.

```
terraform apply
```

### terraform apply will create all the resources that I written in terraform code, once this installation is done it will show us the cluster name as output

I have created a **Jenkins Pipeline** which will deploy **hello-world** application into our **EKS** cluster.

Created an ec2 instance in which I have installed Jenkins and Docker to run our pipelines.

### Jenkins_URL http://3.145.175.147:8080/job/hello-world-deploy/

In this pipeline we have 5 steps:

1. Compling our java code with Maven
2. Building docker image with our Dockerfile
3. Push image to ECR
4. Deploy into K8s.

# Helm Charts

I have a folder called helm in which I have create helm-chart and to deploy into k8s just need to run following command:
```
helm upgrade --install RELEASE_NAME CHART_PATH --set image.tag=ImageTagVersion
e.g
helm upgrade --install hello-world helm/hello-world --set image.tag=18
```
# Autoscaling for K8s Pod
### To make  Autoscaling works need to make sure metric-server pod is running in kube-system namespace. To Deploy or create it just run below command:
``` kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml ```
In  Hello-world helm chart I created HPA (Horizontal Auto Scaling) also which will scale hello world application when cpu utlization reached to certain threshold.
``` kubectl get hpa ```

![image](https://user-images.githubusercontent.com/24311808/146777780-e19b80bc-bd8c-4312-8aed-513f07b14146.png)



# Monitor app with Prometheus and Grafana

### Steps to install Prometheus and Grafana for monitoring our app

I am  deploying these tools with helm chart.

**Deploy Prometheus and Grafana**
1. ``` helm repo add prometheus-community https://prometheus-community.github.io/helm-charts ``` 
this will add  prometheus community helm repo to the system which I will use to install and download charts in EKS cluster.
2. ``` helm install prom prometheus-community/kube-prometheus-stack ```
It will deploy prometheus and their exporter into k8s default namespace.
3. To Monitor Web Application http status blackbox exporter require , this will be deploy through helm charts.
``` helm install blackbox-exporter prometheus-community/prometheus-blackbox-exporter ```
4. To add web application host  need to modify scrape config so it can monitor  web application. Just need to upgrade previous deployed prometheus release with additional scrape config.
``` helm upgrade --install prom prometheus-community/kube-prometheus-stack -f additionalscrape-config.yaml```
Once this is all deployed  make dashboards in grafana, to open Grafana need to expose grafana service as LoadBalancer and open it. Username and Password  take from grafana k8s secrets and decode it for simple text. 

### Grafana Dashboards for our apps, I have created only 1 dashboard to monitor our app.
![image](https://user-images.githubusercontent.com/24311808/146776572-85814234-e568-431f-a661-de9f4c959c00.png)


# Created S3 Resource to store our terraform state
 Thereis a folder called s3 in which code is written to create s3 buckets. Switch to s3 and run ``` terraform init ``` and ``` terraform apply -auto-approve ``` command to create it.

