

# Getting Started with ECS

## Overview

This lab introduces the basics of working with microservices and [ECS](https://aws.amazon.com/ecs/). This includes: setting up the initial ECS cluster, deploying the WordPress and MySQL container from DockerHub, service discovery via Route53 and deployment of the containers with traffic routed through an [ALB](https://aws.amazon.com/elasticloadbalancing/applicationloadbalancer/).

**Note**: You should have containers and Docker knowledge before attempting this lab. You should know the basics of creating a Docker file and a Docker image, and checking in the image into a Docker registry. Otherwise, please complete part 1 and 2 of the [Get Started with Docker tutorial](https://docs.docker.com/get-started/).

The lab architecture is as shown below. WordPress and MySQL containers shall be deployed using AWS Fargate, which allows you to run containers without managing servers. We shall deploy one instance of MySQL container and 2 instances of WordPress. MySQL container registers with a Route53 private hosted zone. WordPress discovers the MySQL via a service lookup to Route53. We will use an Application load balancer to route the requests to WordPress.

![img1]

[img1]:https://github.com/tohwsw/awsecslab/blob/master/Lab21-Getting-Started-with-ECS/img/1-lab-architecture.png

**Note**: MySQL is deployed in containers here to illustrate how Microservices in containers can discover one another. It is not recommended to run databases in containers in a production environment. Docker containers are designed to run stateless applications instead of stateful applications. Database services provided by your cloud provider are the best way to go for production (and that means also staging due to prod/staging parity) databases. Use RDS if you’re on AWS. This will simplify a lot of management tasks such as updating minor versions, handling regular backups and even scaling up.

You'll need to have a working AWS account to use this lab.

