## Overview

Lab 3 will build on Lab 2. In this lab, we shall use AWS CodeDeploy to implement Blue/Green Deployments for AWS Fargate and Amazon ECS.

In [AWS CodeDeploy](https://aws.amazon.com/codedeploy/), blue/green deployments help you minimize downtime during application updates. They allow you to launch a new version of your application alongside the old version and test the new version before you reroute traffic to it. You can also monitor the deployment process and, if there is an issue, quickly roll back.

With this new capability, you can create a new service in AWS Fargate or Amazon ECS that uses CodeDeploy to manage the deployments, testing, and traffic cutover for you. When you make updates to your service, CodeDeploy triggers a deployment. This deployment, in coordination with Amazon ECS, deploys the new version of your service to the green target group, updates the listeners on your load balancer to allow you to test this new version, and performs the cutover if the health checks pass.

## 12. Create a new Task Definition for ColorTeller

On your laptop, we will use the AWS CLI to create a new task definition for colorteller. This updates the colorteller image to output the color "green".

Copy the content below and save it as **colorteller2.json**. Make sure to change account id 284245693010 to your own.

```
{
  "executionRoleArn": "arn:aws:iam::284245693010:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "environment": [
        {
          "name": "COLOR",
          "value": "green"
        }
      ],
      "name": "colorteller",
      "image": "284245693010.dkr.ecr.ap-southeast-1.amazonaws.com/colorteller:latest",
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/fargate",
          "awslogs-region": "ap-southeast-1",
          "awslogs-stream-prefix": "colorteller"
        }
      },
      "memory": 512,
      "cpu": 256,
      "essential": true,
      "portMappings": [
        {
          "containerPort": 8080,
          "hostPort": 8080
        }
      ]
    }
  ],
  "family": "colorteller_fargate_task",
  "memory": "512",
  "cpu": "256",
  "networkMode": "awsvpc",
  "requiresCompatibilities": [
    "FARGATE"
  ]
}

```

Next register the task definitions with ECS. Make sure to run the command in the folder container **colorteller2.json**.

  

    aws ecs register-task-definition --cli-input-json file://colorteller2.json



## 13. Trigger a CodeDeploy blue/green deployment


You now need to update your Amazon ECS service to use the latest revision of your task definition. This will trigger a CodeDeploy blue/green deployment.

1.  Open the Amazon ECS console at [https://console.aws.amazon.com/ecs/](https://console.aws.amazon.com/ecs/).
    
2.  Choose the Amazon ECS cluster where you’ve deployed your Amazon ECS service.
    
3.  Select the check box next to your **colorteller-service** service.
    
4.  Choose **Update** to open the **Update Service** wizard.

5.  Under **Configure service**, for **Task Definition**, choose **2 (latest)** from the **Revision** drop-down list.
        
6.  Choose **Next step**.

7.  Skip **Configure deployments**. Choose **Next step**.
    
8.  Skip **Configure network**. Choose **Next step**.
    
9.  Skip **Set Auto Scaling (optional)**. Choose **Next step**.
    
10.  Review the changes, and then choose **Update Service**.
    
11.  Choose **View Service**.

You are now be taken to the Deployments tab of your service where you can see details about your blue/green deployment.

You can click the deployment ID to go to the details view for the CodeDeploy deployment.

From there you can see the deployments status:

![img3]

[img3]: https://github.com/tohwsw/awsecslab/blob/master/Lab23-BlueGreen-Deployment-with-CodeDeploy/img/3-deployment2.png

If you notice issues, you can stop and roll back the deployment. This shifts traffic back to the original (blue) task set and stops the deployment.

By default, CodeDeploy waits one hour after a successful deployment before it terminates the original task set. You can use the AWS CodeDeploy console to shorten this interval. After the task set is terminated, CodeDeploy marks the deployment complete.

Congrats! You have achieved blue-green deployment with ECS service.


## Lab Clean Up

Below are the high level steps for executing the cleanup.

Go to ECS console to stop all running tasks and remove services.

Go to ECS console to delete **EcsLabPublicCluster**

Go to CloudFormation console to delete the **ecsworkshopstack** stack. This removes the VPC created.

Remove the ecslab namespace in Cloud Map.

Remove the roles created in IAM
- ecstaskexecutionrole
- ecsCodeDeployRole

Delete the code deploy application

Delete the ALB **EcsAlbLab** and **EcsAlbLabInternal** and the target groups.


> Written with [StackEdit](https://stackedit.io/).

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTUwNDE0NTY1MSwxNjYwODQ4NzEwLDE4MT
MwMTMyNCwxNzAxNzA0MzY1LDczMDk5ODExNl19
-->