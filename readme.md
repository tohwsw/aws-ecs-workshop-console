
Lab 2 will build on Lab 1.

## 7. Creating the ALB

**External ALB**

We need an Application Load Balancer [ALB](https://aws.amazon.com/elasticloadbalancing/applicationloadbalancer/) to route traffic to our ColorGateway endpoints. An ALB lets you direct traffic between different endpoints and in this lab, we'll use it to direct traffic to the containers.

To create the ALB, navigate to the [EC2 Console](https://console.aws.amazon.com/ec2/v2/home?#LoadBalancers:sort=loadBalancerName), and select **Load Balancers** from the left-hand menu. Choose **Create Load Balancer**. Create an Application Load Balancer:

![img1]

[img1]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-LoadBalancer.png

Name your ALB **EcsLabAlb** and add an HTTP listener on port 80:

![img2]

[img2]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-LoadBalancer2.png

**Note**: in a production environment, you should also have a secure listener on port 443. This will require an SSL certificate, which can be obtained from [AWS Certificate Manager](https://aws.amazon.com/certificate-manager/), or from your registrar/CA. For the purposes of this lab, we will only create the insecure HTTP listener. DO NOT RUN THIS IN PRODUCTION.

Next, select your VPC and we need at least two subnets for high availability. Make sure to choose the VPC that was used in Lab 1.

![img3]

[img3]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/albaz.png

Click **Next**, and create a new security group (sgecslabloadbalancer) with the following rule:

|Ports| Protocol| Source|
|-----|-------|------|
|80 |tcp| 0.0.0.0/0|

Continue to the next step: **Configure Routing**. For this initial setup, we're just adding a dummy health check on /. We'll add specific health checks for our service endpoints when we register them with the ALB.

![img4]

[img4]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-LoadBalancer4.png

Click through the "Next:Register targets" step, and continue to the **Review** step. If your values look correct, click **Create**.

**Internal ALB**

We create another internal Application Load Balancer [ALB](https://aws.amazon.com/elasticloadbalancing/applicationloadbalancer/) to route traffic the ColorTeller endpoints.

To create the ALB, navigate to the [EC2 Console](https://console.aws.amazon.com/ec2/v2/home?#LoadBalancers:sort=loadBalancerName), and select **Load Balancers** from the left-hand menu. Choose **Create Load Balancer**. Create an Application Load Balancer:

![img111]

[img111]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-LoadBalancer.png

Name your ALB **EcsLabAlbInternal** and add an HTTP listener on port 80:

![img222]

[img222]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/ecslabalbinternal.png

Next, select your VPC and we need at least two subnets for high availability. Make sure to choose the VPC that was used in Lab 1.

![img333]

[img333]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/albaz.png

Click **Next**, and choose **sgecslabloadbalancer** security group.

Continue to the next step: **Configure Routing**. For this initial setup, we're just adding a dummy health check on /. We'll add specific health checks for our service endpoints when we register them with the ALB.

![img444]

[img444]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/ecslabalbinternalrouting.png

Click through the "Next:Register targets" step, and continue to the **Review** step. If your values look correct, click **Create**.


## 8. Set up IAM service roles


Because you will be using AWS CodeDeploy to handle the deployments of your application to Amazon ECS, AWS CodeDeploy needs permissions to call Amazon ECS APIs, modify your load balancers, invoke Lambda functions, and describe CloudWatch alarms. Before you create an Amazon ECS service that uses the blue/green deployment type, you must create the AWS CodeDeploy IAM role (**ecsCodeDeployRole**).

To create an IAM role for AWS CodeDeploy

1.  Open the IAM console at [https://console.aws.amazon.com/iam/](https://console.aws.amazon.com/iam/).
    
2.  In the navigation pane, choose **Roles**, **Create role**.
    
3.  For Select type of trusted entity section, choose **AWS service.**
    
4.  For Choose the service that will use this role, choose **CodeDeploy**.
    
5.  For Select your use case, choose **CodeDeploy**, Next: **Permissions**.
    
6.  Choose **Next: Tags**.
    
7.  For Add tags (optional), you can add optional IAM tags to the role. Choose **Next:Review** when finished.
    
8.  For Role name, type **ecsCodeDeployRole**, enter an optional description, and then choose **Create role**.

9. Click in the **ecsCodeDeployRole** to view it's properties. 
 
10. In the Permissions policies section
-   Choose Attach policies.
    
-   To narrow the available policies to attach, for Filter, type **AWSCodeDeployRoleForECS**
    
-   Check the box to the left of the AWS managed policy and choose Attach policy.
    
-  Choose **Trust Relationships**, Edit **trust relationship**.
    

4  Verify that the trust relationship contains the following policy. If the trust relationship matches the policy below, choose Cancel. If the trust relationship does not match, copy the policy into the Policy Document window and choose **Update Trust Policy**.

```   
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "codedeploy.amazonaws.com"
        ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

5  Since the tasks in your Amazon ECS service using the blue/green deployment type require the use of the task execution role or a task role override, then you must add the **iam:PassRole** permission for each task execution role or task role override to the AWS CodeDeploy IAM role as an inline policy.

Follow the substeps below to create an inline policy.
    
-  Open the IAM console at [https://console.aws.amazon.com/iam/](https://console.aws.amazon.com/iam/).
    
-  Search the list of roles for **ecsCodeDeployRole**.
    
-  In the Permissions policies section, choose **Add inline policy**.
    
-  Choose the JSON tab and add the following policy text. Specify the full ARN of your task execution role or task role override.

   
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": [
        "arn:aws:iam::<aws_account_id>:role/<ecsTaskExecutionRole_or_TaskRole_name>"
      ]
    }
  ]
}

```

6  Choose Review policy
    
7  For Name, type **ecsTaskExecutionPolicy** name for the added policy and then choose **Create policy**.
The **ecsCodeDeployRole** should look like the below.

![img111]

[img111]: https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab3-BlueGreen-Deployment-with-CodeDeploy/img/3-IAMrole.png

## 9. Create the ColorTeller Service

After you have registered a task for your account, you can create a service for the registered task in your cluster. For this example, we will create a service.

Open the ECS dashboard

Choose the colorteller web Task Definition you created in the previous section. Choose Actions -> Create Service.

![img5]

[img5]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-colorteller.png

Configure the service to be Fargate as follows:

![img6]

[img6]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-colorteller2.png

![img666]

[img666]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-wordpress3.png

Next configure the network by selecting the VPC and the 2 subnets.

![img7]

[img7]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-colorteller3.png

Click on the colort--XXX security group to add a new rule for colorteller at port 8080.

![img8]

[img8]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-colorteller4.png

Choose the Load Balancing option as **Application Load Balancer** For Load Balancer name, choose **EcsLabAlbInternal**. Click Add to load balancer.

![img17]

[img17]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/colortelleralb.png

![img18]

[img18]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/colortelleralb2.png

![img19]

[img19]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/colortelleralb3.png


On the Service Discovery section, enter the namespace as **ecslab**.

![img9]

[img9]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-colorteller5.png

Leave the Auto Scaling as **None**. Click Next Step.

![img10]

[img10]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-mysql6.png

On the review page, click Create Service.

**Note**: If the service creation fails the first time, it could be due to the latency in creating the hosted zone on Route53. Retry the service creation by going back to the previous screens and redoing the steps again.

Make sure that the service come up and running.
Go to [CloudWatch Console](https://console.aws.amazon.com/cloudwatch/home) to view the logs

![img11]

[img11]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-colorteller7.png

Go to [Route53 Console](https://console.aws.amazon.com/route53/home) to view the **colorteller-service.ecslab** entry in the private DNS host.

![img12]

[img12]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-route53.png

## 10. Create the ColorGateway Service

We will use the Task Definition created earlier to create the ECS service.
Choose the ColorGateway Task Definition you created in the previous section. Choose Actions > Create Service.

![img13]

[img13]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-colorgw.png

Configure the service to be Fargate as follows:

![img14]

[img14]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-colorgw2.png

![img15]

[img15]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-wordpress3.png

Next configure the network by selecting the VPC and the 2 subnets.

![img16]

[img16]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-colorgw4.png

Add 8080 to the colorg-XXX security group.

![img166]

[img166]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-colorgw41.png

Choose the Load Balancing option as **Application Load Balancer** For Load Balancer name, choose **EcsLabAlb**. Click Add to load balancer.

![img17]

[img17]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-colorgw5.png

![img18]

[img18]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-colorgw6.png

![img19]

[img19]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-colorgw7.png

On the Service Discovery section, uncheck the option. Click **Next Step**.

![img20]

[img20]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-wordpress8.png

Leave the Auto Scaling as **None**. Click **Next Step**.

![img21]

[img21]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-wordpress9.png

On the review page, click **Create Service**.


Make sure that the service come up and running.

Go to [CloudWatch Console](https://console.aws.amazon.com/cloudwatch/home) to view the logs under the log group **/ecs/fargate**

![img22]

[img22]:https://github.com/tohwsw/aws-ecs-workshop/blob/master/Lab2-Create-Service-with-FarGate/img/2-colorgw10.png


## 11. Testing our service deployments from the console and the ALB
  

We can also test from the ALB itself. To find the DNS A record for your ALB, navigate to the EC2 Console -> **Load Balancers** -> **Select your Load Balancer**. Under **Description**, you can find details about your ALB, including a section for **DNS Name**. You can enter this value in your browser, and append the endpoint of your service, to see your ALB and ECS Cluster in action.You should be able to see the following output in the browser. For example http://ecslabalb-1194182192.ap-southeast-1.elb.amazonaws.com/color

```
{"color":"blue", "stats": {"blue":1}}
```

## That's a wrap!

Congratulations! You've deployed an ECS Cluster with a microservice application!

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM5ODQ5ODE0MywtNDQ3NjEwNzQ3LDE0Mj
UwODg2NjAsMTIxODQ5MTY1LDE3NTIzODQyNDMsNzMwOTk4MTE2
XX0=
-->