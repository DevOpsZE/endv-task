Provision an EKS cluster with Terraform

Create a folder and inside create main.tf

Needed for Terraform AWS Provider:
export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)
export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)

terraform init && terraform validate

terraform plan

terraform apply

Test the connection with the cluster by using that file with:
KUBECONFIG=./kubeconfig_my-cluster kubectl get pods --all-namespaces

**If you prefer to not prefix the KUBECONFIG environment variable to every command, you can export it with:
export KUBECONFIG="${PWD}/kubeconfig_my-cluster"

Deploying a simple nginx app:
create a Deployment YAML with nginx

submit the definition to the cluster:
KUBECONFIG=./kubeconfig_my-cluster kubectl apply -f deployment.yaml

route live traffic to the Pod -> create Service of type: LoadBalancer to expose your Pods
1. create service-loadbalancer.yaml
2. KUBECONFIG=./kubeconfig_my-cluster kubectl apply -f service-loadbalancer.yaml

Routing traffic into the cluster with the ALB Controller

Add the EKS repository to Helm:
helm repo add eks https://aws.github.io/eks-charts

Install the TargetGroupBinding CRDs:
KUBECONFIG=./kubeconfig_my-cluster kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller//crds?ref=master"

Install the AWS Load Balancer controller w/o iamserviceaccount:
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster-name

**verify withand get URL:
KUBECONFIG=./kubeconfig_my-cluster kubectl get svc

to verify running MySQL database instance:
aws rds describe-db-instances --filters "Name=engine,Values=mysql" --query "*[].[DBInstanceIdentifier,Endpoint.Address,Endpoint.Port,MasterUsername]"

to verify nginx exposure of serive:
curl http://ad553177d9ae542d2ab92a215ecc7797-1908648400.us-east-1.elb.amazonaws.com:8080/