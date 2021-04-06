## Install AppMesh controller
Follow [install controllere](https://github.com/aws/eks-charts/tree/master/stable/appmesh-controller) here. 

## Create service accounts
```bash
curl -o envoy-iam-policy.json https://raw.githubusercontent.com/aws/aws-app-mesh-controller-for-k8s/master/config/iam/envoy-iam-policy.json

# Create an IAM policy for the proxies from the policy document
aws iam create-policy \
    --policy-name EnvoyControllerIAMPolicy \
    --policy-document file://envoy-iam-policy.json

# Create an IAM role and service account for the application namespace
eksctl create iamserviceaccount --cluster eksworkshop-observ \
    --namespace appmesh-bookinfo \
    --name bookinfo-details \
    --attach-policy-arn arn:aws:iam::$ACCOUNT_ID:policy/EnvoyControllerIAMPolicy  \
    --override-existing-serviceaccounts \
    --approve


eksctl create iamserviceaccount --cluster eksworkshop-observ \
    --namespace appmesh-bookinfo \
    --name bookinfo-ratings \
    --attach-policy-arn arn:aws:iam::$ACCOUNT_ID:policy/EnvoyControllerIAMPolicy  \
    --override-existing-serviceaccounts \
    --approve


 eksctl create iamserviceaccount --cluster eksworkshop-observ \
    --namespace appmesh-bookinfo \
    --name bookinfo-reviews \
    --attach-policy-arn arn:aws:iam::$ACCOUNT_ID:policy/EnvoyControllerIAMPolicy  \
    --override-existing-serviceaccounts \
    --approve


 eksctl create iamserviceaccount --cluster eksworkshop-observ \
    --namespace appmesh-bookinfo \
    --name bookinfo-productpage \
    --attach-policy-arn arn:aws:iam::$ACCOUNT_ID:policy/EnvoyControllerIAMPolicy  \
    --override-existing-serviceaccounts \
    --approve

 eksctl create iamserviceaccount --cluster eksworkshop-observ \
    --namespace appmesh-bookinfo \
    --name envoy-proxies \
    --attach-policy-arn arn:aws:iam::$ACCOUNT_ID:policy/EnvoyControllerIAMPolicy  \
    --override-existing-serviceaccounts \
    --approve
```

## Cleanup service accounts
```
eksctl delete iamserviceaccount --cluster eksworkshop-observ \
    --namespace appmesh-bookinfo \
    --name envoy-proxies

    eksctl delete iamserviceaccount --cluster eksworkshop-observ \
    --namespace appmesh-bookinfo \
    --name bookinfo-details 

      eksctl delete iamserviceaccount --cluster eksworkshop-observ \
    --namespace appmesh-bookinfo \
    --name bookinfo-ratings 

      eksctl delete iamserviceaccount --cluster eksworkshop-observ \
    --namespace appmesh-bookinfo \
    --name bookinfo-reviews 

      eksctl delete iamserviceaccount --cluster eksworkshop-observ \
    --namespace appmesh-bookinfo \
    --name bookinfo-productpage 


    eksctl delete iamserviceaccount --cluster eksworkshop-observ \
    --namespace appmesh-system \
    --name appmesh-controller 

```

## Create App Mesh
```
kubectl apply -f bookinfo/networking/bookinfo-namespace.yaml
kubectl apply -f bookinfo/networking/bookinfo-mesh.yaml
```

## Create App Mesh components
```
kubectl apply -f bookinfo/networking/virtual-nodes.yaml
kubectl apply -f bookinfo/networking/virtual-router.yaml
kubectl apply -f bookinfo/networking/virtual-services.yaml
```

