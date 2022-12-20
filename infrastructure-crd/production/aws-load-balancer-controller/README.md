# AWS Load Balancer controller

Installs [AWS Load Balancer controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/) to help manage Elastic Load Balancers for a Kubernetes cluster. 
* It satisfies Kubernetes Ingress resources by provisioning Application Load Balancers.
* It satisfies Kubernetes Service resources by provisioning Network Load Balancers.


## [Installation](https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/deploy/installation/)

### [Prerequisites](https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/deploy/installation/#iam-permissions)
```sh
# Create IAM OIDC provider
eksctl utils associate-iam-oidc-provider \
    --region eu-central-1 \
    --cluster production-frankfurt \
    --approve

# Download IAM policy for the AWS Load Balancer Controller
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.1.2/docs/install/iam_policy.json


# Create an IAM policy called AWSLoadBalancerControllerIAMPolicy
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json


# Create a IAM role and ServiceAccount for the AWS Load Balancer controller, use the ARN from the step above
kubectl create namespace aws-load-balancer-controller

eksctl create iamserviceaccount \
--cluster=production-frankfurt \
--namespace=aws-load-balancer-controller \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::709278547331:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--approve
```


### [Installation with Helm](https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/deploy/installation/#add-controller-to-cluster)
```sh
# Add helm repository
helm repo add eks https://aws.github.io/eks-charts
helm repo update

# Install the TargetGroupBinding CRDs
kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller//crds?ref=master"

# Install the helm chart
helm install \
  -f values.yaml \
  -n aws-load-balancer-controller \
  aws-load-balancer-controller eks/aws-load-balancer-controller
```


## Upgrade

```sh
helm upgrade -f values.yaml aws-load-balancer-controller eks/aws-load-balancer-controller
```