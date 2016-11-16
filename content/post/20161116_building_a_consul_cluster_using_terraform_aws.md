+++
date = "2016-11-16T10:26:33+01:00"
tags = ["Consul", "terraform"]
title = "Building a Consul cluster using Terraform/AWS"
+++

<img style="float: right;" width="150" src="https://www.consul.io/assets/images/logo_large-475cebb0.png">
[Consul](http://Consul.io) has multiple components, but as a whole, it is a tool for discovering and configuring services in your infrastructure. It provides several key features:

* Service Discovery
* Health Checking
* Key/Value Store
* Multi Datacenter

For more information on [Consul](http://Consul.io) itself please have a look in the excellent documentation.

* [Consul Intro](https://www.consul.io/intro/index.html)
* [Consul documentation](https://www.consul.io/docs/index.html)

## Is it really easy?
Setting up a [Consul](http://Consul.io) cluster seems easy, just follow the many tutorials out there and you will have a Consul cluster running in a few steps on your local machine...

But hey.. **what if you need to deploy this cluster on an AWS environment? How do
you create the cluster and how can you make sure it is always available?**

This simple write up is just an example to give you an idea how this Consul cluster can be created and provisioned by using Terraform only. The goal is to have a cluster using the official Docker images provided by Consul itself running on EC2 nodes.

## Creating a Consul cluster
The principle is not that hard... Consul nodes can discover each other based on IP Address. If you feed the Consul cluster members with IP Addresses that are part of the cluster you are good to go. In the example case we are going to start a number of Consul cluster members. The first node will be unable to form a cluster but if the second node starts up it will get the ip from the first node and the the first node will then know the ip of the second one.. etc.. So if you start up more than 2 nodes you will be good to go.

<center>

    +------+   +------+  +------+  +------+  +------+
    |Consul|   |Consul|  |Consul|  |Consul|  |Consul|
    |Node 1|   |Node 2|  |Node 3|  |Node 4|  |Node n|
    +------+   +------+  +------+  +------+  +------+
    < Find each other based on ip address >

</center>

The power is in the user-data script that is used for bootstrapping the Consul cluster nodes. In the example case they will find each other based on a query using `aws ec2 describe-instances` that will find nodes with a specific name, and from those identified nodes we will extract the IP Addresses that will be used to joint the Consul cluster. You can always modify the query to your own needs off course. The user-data script is used in the launch configuration.

So enough talk... lets start :)

**The given examples are intentionally kept simple!! So you will need to tweak your Terraform code according to your needs**

## Step 1: Define a launch Configuration

The core component of our Consul cluster is the launch configuration. This launch configuration determines what user-data file needs to be executed when launching a new instance.

``` terraform
resource "aws_launch_configuration" "consul-cluster" {

  /*
    in this case we use a docker optimized ami, because we are going
    to use the official Consul docker image as a starter.
    You can always use an ami on which you install docker manually!
  */
  image_id  = "docker-optimized-ami-0123456789"

  /*
    the user-data script that holds the magic
  */
  user_data = "${data.template_file.consul-cluster.rendered}"
  instance_type = "t2.micro"

  /*
    make sure you open the correct ports so the Consul nodes can
    discover each other the actual security group is not shown
  */
  security_groups = ["sg-0123456789"]
  key_name = your-deploy-key

  /*
    us a policy which grants read access to the EC2 api
  */
  iam_instance_profile = "arn:aws:iam::0123456789:read_ec2_policy/ec2"
}

/*
    The template file used for the user-data
*/
data "template_file" "consul-cluster" {
  template = "user-data-consul-cluster.tpl")}"
  vars {
    // the name must match the Name tag of the autoscaling group
    consul_cluster_name = "consul-cluster-member"

    // the number of instances that need to be in the cluster to be healthy
    consul_cluster_min_size = 3
  }
}
```

## Step 2: Create the template file

The user-data file is going to query AWS using the aws describe-instances api and will return ec2 nodes that have a matching name using the \-\-filters option. `'Name=tag:Name,Values=${consul_cluster_name}'`

All the retrieved instances are then queried for their private ip and the values are stored in a list. After completing the list the instance ip for the current machine is removed.

A Consul specific join string is composed and provided to the docker image. This enables the Consul docker image to check for available servers when starting.

```bash
// File: user-data-consul-cluster.tpl

#!/bin/bash -ex
exec > >(tee /var/log/user-data.log|logger -t user-data -s 2>/dev/console) 2>&1

echo 'installing additional software'
for i in {1..5}
do
    yum install -y aws-cli && break || sleep 120
done

################################################################################
# Running Consul
################################################################################
# get current instance ip
INSTANCE_IP=$(curl http://169.254.169.254/latest/meta-data/local-ipv4)

# get list of available Consul servers; based on Name (value) tag!
IP_LIST=$(aws ec2 describe-instances --region us-east-1 \
--filters 'Name=tag:Name,Values=${consul_cluster_name}' \
'Name=instance-state-name,Values=running' \
--query "Reservations[*].Instances[*].PrivateIpAddress" \
--output=text)

# remove the current instance ip from the list of available servers
IP_LIST="$${IP_LIST/$$INSTANCE_IP/}"

# remove duplicated spaces, \r\n and replace space by ','
IP_LIST=$(echo $$IP_LIST | tr -s " " | tr -d '\r\n' | tr -s ' ' ',')

# create join string
for i in $(echo $IP_LIST | sed "s/,/ /g")
do
    JOIN_STRING+="-retry-join $i "
done

# - run Consul
docker run -d --net=host \
-e 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true}' \
--name consul-server consul:latest \
agent -server -bind=$INSTANCE_IP $JOIN_STRING \
-bootstrap-expect=${consul_cluster_min_size} -ui -client 0.0.0.0
# ------------------------------------------------------------------------------
```

## Step 3: Create an autoscaling group

``` terraform
/*
    creates an autoscaling group so servers are created when needed
*/
resource "aws_autoscaling_group" "consul-cluster" {
  min_size = 3
  max_size = 5
  desired_capacity = 3
  min_elb_capacity = 3

  launch_configuration = "${aws_launch_configuration.consul-cluster.name}"
  load_balancers = ["${aws_elb.consul.id}"]

  tag {
    key = "Name"
    /*
      note: this is the value that is being searched for in the user-data
    */
    value = "consul-cluster-member"
    propagate_at_launch = true
  }
}
```

## Step 4: Create an ELB as frontend for the Consul cluster

``` terraform
resource "aws_elb" "consul-cluster" {
  name = "consul-cluster"
  subnets = ["sn-0123456789"]
  security_groups = ["sg-0123456789"]

  listener {
    instance_port = 8300
    instance_protocol = "tcp"
    lb_port = 8300
    lb_protocol = "tcp"
  }

  listener {
    instance_port = 8301
    instance_protocol = "tcp"
    lb_port = 8301
    lb_protocol = "tcp"
  }

  listener {
    instance_port = 8302
    instance_protocol = "tcp"
    lb_port = 8302
    lb_protocol = "tcp"
  }

  listener {
    instance_port = 8400
    instance_protocol = "tcp"
    lb_port = 8400
    lb_protocol = "tcp"
  }

  listener {
    instance_port = 8500
    instance_protocol = "http"
    lb_port = 8500
    lb_protocol = "http"
  }

  listener {
    instance_port = 8600
    instance_protocol = "tcp"
    lb_port = 8600
    lb_protocol = "tcp"
  }

  health_check {
    target = "HTTP:8500/v1/status/leader"
    healthy_threshold = 2
    unhealthy_threshold = 2
    interval = 30
    timeout = 5
  }
}
```

When putting all the pieces together you should now have a running Consul cluster!
