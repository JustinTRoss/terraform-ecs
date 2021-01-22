# Infrastructure/AWS:

## VPC: Virtual Private Cloud

_A virtual network in which you build out your architecture._

A VPC is much like a piece of real estate:

- You can secure the perimeter of the property as much or as little as you like, putting up a fortified wall, a picket fence, or no barrier at all, possibly putting up a wall on one side, and leaving an open section out front to enter from the street.
- Inside this perimeter security, you can leave sections with no additional security, like you might a garden or a yard. The security you choose here likely depends on your perimeter security. If you have deer around and no security externally, you might opt to fence your garden.
- Inside this perimeter security, you can secure sections even further, possibly in a nested fashion, as you might a house, a safe room within that house, or even a safe within that safe room.

Enough with the real estate analogy...

In your VPC, you'll define a private IP range, specified in CIDR notation. This will look something like 10.0.0.0/16. That last portion, "/16", specifies the number of IPs you want to include. Everything before the "/16" specifies what IP to start at. "/32" indicates 1 IP, "/31" indicates 2 IPS, "/30" is 4, "/29" is 8, and so on, doubling each decrement from 32. Our "/16" is equivalent to 65,536.

This private IP range is an internal reference used to divide up your property. As private IPs, no external addresses can reference them, and many different people will have the same addresses.

In our real estate analogy, these private IPs might be equivalent to how we use names to separate our house and our garden, or further our living room from our kitchen. We can break up that private IP range into a bunch of smaller ranges, which we assign to services within our VPC.

Each subnet of IPs can be assigned its own additional security rules.

### Private Subnet:

_A portion of your VPC designated for access only by services within your VPC._

### Public Subnet:

_A portion of your VPC designated for access by the outside world._

## EBS: AWS Elastic Block Storage

_AWS storage space you can allocate to extend the capabilities of servers you're running. Like an external hard drive for your EC2 instance._

## Routing table:

_A set of rules that define where network traffic is directed. This includes both traffic from your subnet and from your internet gateway._

## AMI: AWS Amazon Machine Image

_An Amazon Machine Image is a special type of virtual appliance that is used to create a virtual machine within the Amazon Elastic Compute Cloud. It serves as the basic unit of deployment for services delivered using EC2._

This is the basic collection of software running on the server. OS, CLIs, etc.

## ECR: AWS Elastic Container Registry

_An AWS service, which allows you to store, manage, share, and deploy your container images and artifacts._

A container image is an executable which includes all the information needed to run a container. What an AMI is to a server, a container image is to a container. An AMI might container the code to run docker, while a container image might include the node runtime and your application code.

## Subnet:

_A range of IPs, typically defined in CIDR notation. Subnets are composed of subnets._

## ECS: AWS Elastic Container Service

_An AWS service, which will manage the lifecycle and scaling of your services, using EC2 hosts and services run in a container, like Docker._

### ECS Service ???

### ECS Task ???

### ECS Container Agent ???

### User_date ???

## NAT: Network Address Translation

_A server that specializes in mapping traffic from resources outside the VPC to private resources within it._

Required in order to allow a private IP to address an IP outside your VPC. This is because the outside world can't reference your private IPs. When your private server requests data from an external server, that external server doesn't know who to send the data back to.

## Load Balancer

_A service that automatically distributes your incoming traffic across multiple targets._

### ELB: AWS Elastic Load Balancing

_It monitors the health of its registered targets, and routes traffic only to the healthy targets. Elastic Load Balancing scales your load balancer as your incoming traffic changes over time._

## IAM: AWS Identity and Access Management

_An AWS service that helps you securely control access to AWS resources. You use IAM to control who is authenticated (signed in) and authorized (has permissions) to use resources._

## Policy:

Used to detail access and permissions for a resource.

## Cloudwatch:

_AWS's default logging option._

Default region for cloudwatch is us-east-1.

## Region:

_The part of the world you want your services to run in._

## Availability zone:

_The specific data-centers your want your services to be run in._

## AWS CLI profile:

_A name used to reference a set of credentials for an AWS account. Default is "default"._

## RDS: AWS Relational Database Service

_An AWS service, which simplifies the setup, operation, and scaling of database infrastructure._

## Security group:

_A set of rules for inbound and outbound traffic, which can be applied to a set of services._

## API Gateway:

_A service designated to broker public interactions with your services._

This can be useful for specifying access requirements, rate limits, compression algorithms, etc.

## Proxy Server:

_A server that fields network requests on behalf of another server._

### Forward Proxy:

_A server setup on the behalf of the requester to send and retrieve traffic on their behalf. For example, to access content forbidden to the requester._

### Reverse Proxy:

_A server setup on behalf of the receiver to broker requests on it's behalf. For example, to disperse traffic among instances to reduce load on any individual server._

## Bastion Host:

_A security-focused server in your VPC, from which you must access the other servers within your VPC. This allows you to minimize security surface area._
