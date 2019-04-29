## Overview

Lab 3 will build on Lab 2. In this lab, we shall use AWS CodeDeploy to implement Blue/Green Deployments for AWS Fargate and Amazon ECS.

In [AWS CodeDeploy](https://aws.amazon.com/codedeploy/), blue/green deployments help you minimize downtime during application updates. They allow you to launch a new version of your application alongside the old version and test the new version before you reroute traffic to it. You can also monitor the deployment process and, if there is an issue, quickly roll back.

With this new capability, you can create a new service in AWS Fargate or Amazon ECS that uses CodeDeploy to manage the deployments, testing, and traffic cutover for you. When you make updates to your service, CodeDeploy triggers a deployment. This deployment, in coordination with Amazon ECS, deploys the new version of your service to the green target group, updates the listeners on your load balancer to allow you to test this new version, and performs the cutover if the health checks pass.

## 12. Create a new Task Definition for Wordpress

On your laptop, we will use the AWS CLI to create a new task definition for Wordpress. This updates the Wordpress image from 4.6 to 4.7 so that we can update the ECS service.

Copy the content below and save it as **wordpress2.json**. Make sure to change the arn **arn:aws:iam::284245693010:role/ecsTaskExecutionRole** of **ecsTaskExecutionRole** to your own. The arn can be found in the role on the IAM console.

```
   {
  "executionRoleArn": "arn:aws:iam::284245693010:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "environment": [
        {
          "name": "WORDPRESS_DB_HOST",
          "value": "mysql-service.ecslab"
        },
        {
          "name": "WORDPRESS_DB_PASSWORD",
          "value": "password"
        }
      ],
      "name": "wordpress",
      "image": "wordpress:4.7",
      "memory": 512,
      "cpu": 256,
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/fargate",
          "awslogs-region": "ap-southeast-1",
          "awslogs-stream-prefix": "wordpress"
        }
      },
      "essential": true,
      "portMappings": [
        {
          "containerPort": 80,
          "hostPort": 80
        }
      ]
    }
  ],
  "family": "wordpress_fargate_task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": [
    "FARGATE"
  ],
  "memory": "512",
  "cpu": "256"
}
```

Next register the task definitions with ECS. Make sure to run the command in the folder container **wordpress2.json**.

  

    $aws ecs register-task-definition --cli-input-json file://wordpress2.json



## 13. Trigger a CodeDeploy blue/green deployment


You now need to update your Amazon ECS service to use the latest revision of your task definition. This will trigger a CodeDeploy blue/green deployment.

1.  Open the Amazon ECS console at [https://console.aws.amazon.com/ecs/](https://console.aws.amazon.com/ecs/).
    
2.  Choose the Amazon ECS cluster where you’ve deployed your Amazon ECS service.
    
3.  Select the check box next to your **wordpress-service** service.
    
4.  Choose **Update** to open the **Update Service** wizard.
    
    1.  Under **Configure service**, for **Task Definition**, choose **2 (latest)** from the **Revision** drop-down list.
        
5.  Choose **Next step**.
6.  Skip **Configure deployments**. Choose **Next step**.
    
7.  Skip **Configure network**. Choose **Next step**.
    
8.  Skip **Set Auto Scaling (optional)**. Choose **Next step**.
    
9.  Review the changes, and then choose **Update Service**.
    
10.  Choose **View Service**.

You are now be taken to the Deployments tab of your service where you can see details about your blue/green deployment.

![img2]

[img2]: https://github.com/tohwsw/awsecslab/blob/master/Lab23-BlueGreen-Deployment-with-CodeDeploy/img/3-deployment.png

You can click the deployment ID to go to the details view for the CodeDeploy deployment.

From there you can see the deployments status:

![img3]

[img3]: https://github.com/tohwsw/awsecslab/blob/master/Lab23-BlueGreen-Deployment-with-CodeDeploy/img/3-deployment2.png

If you notice issues, you can stop and roll back the deployment. This shifts traffic back to the original (blue) task set and stops the deployment.

By default, CodeDeploy waits one hour after a successful deployment before it terminates the original task set. You can use the AWS CodeDeploy console to shorten this interval. After the task set is terminated, CodeDeploy marks the deployment complete.

Congrats! You have achieved blue-green deployment with ECS service.

## Post Lab Questions

- Would you be able to ssh into the container instance using Fargate mode?

- How would you promote the same container image across different environments (or ECS clusters). How do you externalize the variables(such as database DNS) in each of the environment?

- What are the activities that you put in a container DevOps pipeline?

- The MySQL password is stated in the clear in the Task Definition. How would you secure this?

- If you stop and start the MySQL service in the lab, the table data will be lost. Why is that so? How do you persist the data in ECS?

- Compare the time for deployment for a new application version for ECS vs Elastic Beanstalk. Which is faster? What is that so?

## Lab Clean Up

Below are the high level steps for executing the cleanup.

Go to ECS console to delete **EcsLabPublicCluster**

Go to CloudFormation console to delete the **ecsworkshopstack** stack.

Remove the private hosted zone in Route53.
```
$ aws servicediscovery list-services
$ aws servicediscovery delete-service --id <service id>
$ aws servicediscovery list-namespaces
$ aws servicediscovery delete-namespace --id <namespace id>
```
Remove the roles created in IAM
- ecslabinstanceprofile
- ecstaskexecutionrole
- ecsCodeDeployRole

Delete the **sgecslabpubliccluster** security group

Delete the code deploy application

Delete the ELB **EcsAlbLab** and the target groups.


> Written with [StackEdit](https://stackedit.io/).

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTUwNDE0NTY1MSwxNjYwODQ4NzEwLDE4MT
MwMTMyNCwxNzAxNzA0MzY1LDczMDk5ODExNl19
-->