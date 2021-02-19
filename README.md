AWS Transit Gateway - Partial Mesh (traffic is isolated based on the environment) with Terraform
===========================================
This project gives an example of the usage of AWS Transit Gateway (https://aws.amazon.com/transit-gateway/ product. AWS Transit Gateway provides a way to interconnect multiple VPCs and or accounts in a hub-and-spoke topology.


Architecture
------------
A Transit Gateway relies on Route Tables. By default, a new Route Table is created in the Transit Gateway, which populates with the routing info toward every VPC attached to the gateway (the full mesh scenario). The Terraform code in this project demonstrates a more complex scenario in which traffic is isolated based on the environment. Four VPCs are created, with two subnets each (in separate AZs):

* VPC1-dev:     in the 'dev' environment
* VPC2-dev:     in the 'dev' environment
* VPC3-shared:  in the 'shared' environment
* VPC4-prod:    in the 'prod' environment

Let's assume the 'shared' environment will host shared components, such as proxy services, tools, ... Here are the rules we want our Transit Gateway to implement:

* The shared VPC can access dev and prod VPCs.
* The dev VPCs can access each other, and the shared VPC
* The prod VPCs can only access the shared VPC

To enable such a scenario, three Route Tables are created in the Transit Gateway, one per environment. Which means both dev VPCs attach to the same Route Table, whereas the shared and prod VPCs each attach to their respective Route Table. Each VPC gets a t2.micro Ubuntu instance to validate the network connectivity over ssh and ICMP (ping).


How to test?
------------
* VThe instance in the 'shared' is assigned a public IP so a VPN connection isn't needed.
* You DO need to add your external IP to the "shared" security group to be able to connect to the EC2 instance via SSH (port 22).


What's Next?
------------
* Adding necessary Terraform resources to establish a VPN connection
* Add architecture diagram
* Provide testing detail steps


Prerequisites
-------------
* An AWS account
* Terraform


Usage
-----
* Change ACCESS_KEY and SECRET_KEY values in Variables.tf
* Change the public_key value to a keypair you own. If you don't have one, create a new one using the command "ssh-keygen -f mykey"
* Deploy the setup with:
```sh
$ terraform init
$ terraform plan
$ terraform apply
```
* The public IP of the instance in the 'shared' VPC is printed when deployment ends
* ssh on this instance
```sh
$ ssh -i mykey_private_part ubuntu@$PUBLIC_IP
```
* Check you can ping and ssh any other instance in the other VPCs
* Also check that, from a dev instance (1 & 2) you cannot reach the prod instance (4) and vice-versa.


Cleanup
-----
* Delete all resources
```sh
$ terraform destroy
```
