# ENIConfig Controller

This repository will inplement auto annotating your Kubernete data plane nodes
with a desired `ENIConfig` name. This was originally implemented in the
`amazon-vpc-cni-k8s` project in this pull request - https://github.com/aws/amazon-vpc-cni-k8s/pull/165

## Prerequisites

Before patching the node with the annotation, you will need to change the AWS_VPC_K8S_CNI_CUSTOM_NETWORK_CFG environment variable in the AWS CNI daemonset to true. By default, pods share the same subnet and security groups as the worker node's primary interface. When you set this variable to **true** it causes ipamD to use the security groups and VPC subnet in a worker node's ENIConfig for elastic network interface allocation. You must create an ENIConfig custom resource definition for each subnet that your pods will reside in, and then annotate each worker node to use a specific ENIConfig (multiple worker nodes can be annotated with the same ENIConfig). Worker nodes can only be annotated with a single ENIConfig at a time, and the subnet in the ENIConfig must belong to the same Availability Zone that the worker node resides in.

The worker nodes have to be assigned an IAM role that grants DescribeTags to the EC2 instance.  This is configured by default when you use eksctl to provision a cluster.  

## Deploying

If you use `helm` to deploy applications you can use the
`charts/eniconfig-controller` chart to deploy the controller.

```bash
helm install --set eniConfigName=<name-of-eniconfig-cr> ./charts/eniconfig-controller
```

If you do not use `helm` you can download the config file using the below steps
`wget` or `curl -o` the `configs/eniconfig-controller.yaml` then modify the
`args` to represent the proper `ENIConfig` name. E.G.

```yaml
args:
- --eniconfig-name=<name-of-eniconfig>
```

Then `apply` this config to the cluster.

```bash
kubectl apply -f eniconfig-controller.yaml
```

## Running in Dev

```sh
# assumes you have a working kubeconfig, not required if operating in-cluster
$ go build -o eniconfig-controller .
$ ./eniconfig-controller -kubeconfig=$HOME/.kube/config -eniconfig-name=name-of-eni

## Releasing

To release this project all you have to do is run `make release`.
