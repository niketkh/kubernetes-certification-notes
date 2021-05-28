Logging in to the Amazon Web Services Console
---------------------------------------------

1) Go to https://991845646788.signin.aws.amazon.com/console?region=us-west-2
2) Enter the following credentials created just for your lab session, and click Sign in:
	Account ID or alias: 991845646788
	IAM user name: student
	Password: Ca1_gyYz0V3f
3) Select the US West (Oregon) us-west-2 region using the upper-right drop-down menu on the AWS Management Console


Creating the Elastic Container Repository
-----------------------------------------

1) In the AWS Management Console search bar, enter ECS, and click the Elastic Container Service result under Services
2) Click on Amazon ECR > Repositories in the left sidebar menu
3) Click on the orange Create repository button.
4) In the Repository configuration section, enter ca-container-registry as the Repository name and then click Create repository.
5) Copy the URI of the repository you just created by clicking the two squares icon beside the URI and paste somewhere you can reference later
	991845646788.dkr.ecr.us-west-2.amazonaws.com/ca-container-registry

Building Docker Images
----------------------
CodeBuild is a fully-managed build service. CodeBuild can compile code, run tests, and produce packages you can deploy on any compatible resource. Using CodeBuild in this Lab eliminates the need for you to create a build server or install any software on your local machine. In this Lab Step, you will use CodeBuild to build the Docker images containing the two applications you need to complete the Lab.

1) Search for CodeBuild in the Services search bar to navigate to the CodeBuild Console
2) Click Create build project
3) In the Create build project step area specify the following details leaving the defaults for the rest:

	Project configuration:
	Project name: ecslab-blue-project

	Source:
	Source provider: Amazon S3
	Bucket: Select bucket starting with cloudacademylab-caecslab-
	S3 object key: ecslabblue.zip

	Environment:
	Environment image: Use an image managed by AWS CodeBuild
	Operating system: Ubuntu
	Runtime: Standard
	Image: aws/codebuild/standard:3.0
	Image version: Always use the latest image for this runtime version
	Privileged: Checked
	Service role: Choose an existing service role from your account
	Role name: CodeBuildServiceRole 
	Allow AWS CodeBuild to modify this service role...: Unchecked

	Buildspec:
	Build specification: Use the buildspec.yml in the source code root directory
	Buildspec name: buildspec.yml

	Artifacts:
	Artifacts type: No artifacts
	Cache: No cache

The project source files and Amazon S3 bucket are created for you by the Cloud Academy Lab environment. The project uses Ubuntu Linux with Docker installed as the build environment to create the image. The build spec is a collection of build commands and settings that provide instructions on how to build the Docker image and where to push it once complete. For the purposes of differentiating the images and the applications they contain in this Lab, the build spec also tags the images with distinct names. CodeBuild will tag your blue application image with testblue and your green application image with testgreen due to instructions in the source files.

4) To create your code build project, click Create build project.
5) Click on Edit > Environment
6) Uncheck Allows AWS CodeBuild to modify this service role... if it has been selected
7) Expand Additional configuration and click Add environment variable and enter the following before clicking Update environment
	Name: AWS_ACCOUNT_ID
	Value: 991845646788 (the student AWS Account ID)

The Account ID is used to construct the ECR repository URI during the build process. The AWS region is also required to construct the URI, however CodeBuild includes an AWS_REGION environment variable in build environment by default.

8) Click Start build and click Start build again the form that opens.

This will open a build information page. Once CodeBuild finishes building the Docker image, it executes a Docker push command that stores the image in the ECR repository.

9) Observe the Phase Details section as each step of the build process proceeds and completes. Wait approximately one minute until the Status field shows Succeeded. CodeBuild is building a Docker container image containing one of the custom applications (blue) made for this Lab

10) Move to the Build logs section to view information about the build process. This screen shows up to the last 10000 lines of the build log. The lines in the following image show part of the POST_BUILD stage where the layers of the image are pushed to the repository before finalizing the build

11) Click View entire log to go to CloudWatch logs and view detailed information about every step in the build process. Examining and retaining these logs can be useful for troubleshooting purposes


12) Return to AWS CodeBuild > Build projects and click Create build project

For convenience's sake and for the purposes of this Lab demonstration, you will create both the blue and green application container images at the same time. In a real environment your releases may be weeks or months apart, or several times a day, depending on your development and deployment approach.

13. The steps and details for creating the green project are almost identical to those for the blue project. Enter the following details in the Configure your project section:

	Project configuration:
	Project name: ecslab-green-project
	Source:
	Source provider: Amazon S3
	Bucket: Select bucket starting with cloudacademylab-caecslab-
	S3 object key: ecslabgreen.zip
	Environment:
	Environment image: Use an image managed by AWS CodeBuild
	Operating system: Ubuntu
	Runtime: Standard
	Image: aws/codebuild/standard:3.0
	Image version: Always use the latest image for this runtime version
	Privileged: Checked
	Service role: Choose an existing service role from your account
	Role name: CodeBuildServiceRole 
	Allow AWS CodeBuild to modify this service role...: Unchecked
	Buildspec:
	Build specification: Use the buildspec.yml in the source code root directory
	Buildspec name: buildspec.yml
	Artifacts:
	Artifacts type: No artifacts
	Cache: No cache
	Logs:
	CloudWatch logs - optional: Unchecked
	One important difference for your second CodeBuild project is you will use the ability to enter advanced settings in the initial project creation form.


14. Uncheck Allows AWS CodeBuild to modify this service role before expanding Environment > Additional configuration and click Add environment variable and enter the following:

	Name: AWS_ACCOUNT_ID
	Value: 991845646788 (the student AWS Account ID)


15. Review the project details in the Review section and click Create build project.

16. Click Start build.

17. Wait approximately one minute for the Status field in the Build section to show Succeeded. Examine the build log and details, if you wish. All of the build process information is very similar to the blue project, but the new green container has different unique identifiers.

18. Return to AWS CodeBuild > Build projects and view your two projects. You can initiate a new build and create new containers at any time by selecting a project and clicking Start build. This is not necessary for this Lab, but a useful tool in case you upload a new version of the application to one of your source locations:

So where do your container images go? You are building Docker container images, but where can you view the result of your build operations?

19. Return to Services > ECS and click Amazon ECR > Repositories. Click on ca-container-registry to view your Docker container images:

The application buildspec.yml files tag the container images with testgreen or testblue, depending on the application they contain.

Creating ECS Cluster
--------------------

1. Click on Amazon ECS > Clusters and click Create Cluster:

2. In the Create Cluster section select EC2 Linux + Networking as the cluster template. Click Next step:

3. In the Configure cluster section enter the following details, leaving the other settings as the defaults before clicking Create:

	Cluster name: ecslab-cluster
	EC2 instance type: t2.micro
	Warning!: The Lab environment is restricted to only allow the t2.micro instance type. You must select a t2.micro or the cluster will not launch.

	VPC: Choose the VPC with a name ending in cloudacademylab. The Lab environment creates this VPC specifically for this Lab.
	Subnets: Select all available subnets.
	Auto assign public IP: Enabled
	Security group: Select the Security Group (SG) whose name contains cloudacademylab-UsingECSLabSecurity...
	Container instance IAM role: ecsInstanceRole

4. Observe the cluster creation process on the Launch status screen. Wait until all three steps complete and click View Cluster

	The ECS cluster creation process is a fairly complex operation that utilizes the following Amazon services:

	Elastic Compute Cloud (EC2) - The cluster is powered by EC2 instances and networking features.
	Identity and Access Management - The EC2 instances require roles with defined policies to securely interact with other services.
	CloudFormation - CloudFormation is the mechanism that deploys and manages your cluster as configured.
	Auto Scaling - Though not used in this Lab, it is important to note you can configure Auto Scaling rules and conditions for your cluster.
 				

Creating the Task Definitions
-----------------------------

1. Select Task Definitions. Click Create new Task Definition:

2. In the Create new Task Definition step select EC2 as the launch type. Click Next step

3. Enter a Task Definition Name of ecslab-blue-taskdef. Leave the other settings as default. Scroll down and click Add container in the Container Definitions section

4. Enter the following information in the Standard section of the Add container page. Leave other settings as default.

	Container name: ecslab-blue-container
	Image: <YOURREPOSITORYURI>:testblue, for example - 699069904892.dkr.ecr.us-west-2.amazonaws.com/ca-container-registry:testblue
	Memory Limits (MB): Hard limit 128
	Port mappings: Host port = 0, Container port = 8081
	Scroll down and click Add

The Host port 0, Container port 8081 tells the EC2 instance hosting the containers to look for traffic on any port an application is using. Port 8081 is the traffic port exposed by the Docker container.

5. When you return to the Create a Task Definition page, note that you now have an entry in the Container Definitions section. Click Create

Container definitions are a set of parameters passed to the Docker daemon on a container instance. In this case you have specified the name, image, and memory limit parameters.

6. Verify you see the Created Task Definition successfully message and can now view details about the ecslab-blue-taskdef task definition

7. Return to Task Definitions and click Create new Task Definition again. As with the Docker container images, you must create another Task Definition for your green application. Select the EC2 launch type. Click Next step. 

8. Enter a Task Definition Name of ecslab-green-taskdef. Leave the other settings as default. Click Add container.

9. Enter the following information in the Standard section of the Add container page. Leave other settings as default:

	Container name: ecslab-green-container
	Image: <YOURREPOSITORYURI>:testgreen, for example - 699069904892.dkr.ecr.us-west-2.amazonaws.com/ca-container-registry:testgreen
	Memory Limits (MB): Hard limit 128
	Port mappings: Host port = 0, Container port = 8081
	Click Add.

10. Verify you see your ecslab-green-container in the Container Definitions section. Click Create.

11. Verify you see the Created Task Definition successfully message and can now view details about the ecslab-green-taskdef task definition.

Creating the Target Group and Load Balancer
-------------------------------------------

Using multiple containers is a good practice to avoid a single point of failure and maintain availability. However, making the front-end access point of an application reliant on fixed references to dynamic and scalable cloud resources is not the best idea. In this Lab Step, you will create a target group and load balancer to consolidate access to your resources in one place with one Domain Name Server (DNS) entry. 

1. Search for EC2 in the service search bar to navigate to the EC2 Console

2. Select LOAD BALANCING > Target Groups from the EC2 Dashboard column. 

3. Click Create target group.

A target group is a group of resources, such as container or EC2 instances, registered and grouped together for reference by an Application Load Balancer (ALB).

4. Enter the following information in the Create target group window. Leave other settings as default. Click Next and Create target group:

Target group name: ecslab-targetgroup
VPC: Select the VPC with cloudacademylab in the name.
Health Check Path: /api/

The /api/ path reference is not an essential part of creating every load balancer. It is a specific requirement for your Lab example application.

5. Verify you receive a Successfully created target group message and click Close.

6. Select LOAD BALANCING > Load Balancers from the EC2 Dashboard column.

7. Click Create Load Balancer. 

8. Click Create under the Application Load Balancer.

An ALB is a type of load balancer that functions at the application layer, the seventh layer of the Open Systems Interconnection (OSI) model. It is similar to a classic Elastic Load Balancer (ELB) in that it distributes traffic among targets using a routing algorithm. However, ALBs have the ability to examine content and route traffic accordingly. The essential ALB feature for this Lab is support for containerized applications. ALBs have the ability to select an unused port when scheduling a task and registering a task with a target group using this port.

9. On the Step 1: Configure Load Balancer page, enter the following information. Leave the other settings as default. Click Next: Configure Security Settings:

Name: ecslab-alb
VPC: Select the VPC with cloudacademylab in the name.
Availability Zones: Select all available Availability Zones.

10. Ignore the warning about a secure listener. It is not important for this Lab exercise. Click Next: Configure Security Groups.

11. Choose Select an existing security group and select the SG with cloudacademylab in the name. Click Next: Configure Routing

12. Enter the following information in the Target group and Health checks sections. Leave other settings as default. Click Next: Register Targets:

Target group: Existing target group
Name: ecslab-targetgroup

This Lab shows the Advanced health check settings for informational purposes. You can specify a port for routing purposes and control the health check thresholds. The thresholds determine how quickly a target registers as healthy or unhealthy.

13. There are no targets available in Step 5: Register Targets. Click Next: Review. As your ECS services launch tasks and container instances, they will dynamically register with the target group and load balancer.

14. Review the ALB information in Step 6: Review and click Create.

15. Verify you receive a Successfully created load balancer message. Click Close.


Creating the ECS Services
-------------------------

An ECS service is a mechanism that allows ECS to run and maintain a specified number of instances of a task definition. If any tasks or container instances should fail or stop, the ECS service scheduler launches another instance to replace it. This is similar to Auto Scaling in that it maintains a desired number of instances, but it does not scale instances up or down based on CloudWatch alarms or other Auto Scaling mechanisms. Services behind a load balancer provide a relatively seamless way to maintain a certain amount of resources while keeping a single application reference point. In this Lab Step you will create two services, one for your blue application and one for the green application.

1. Return to Services > EC2 Container Service > Amazon ECS > Clusters and click on the ecslab-cluster

2. Click the Create button on the Services tab

3. Enter the following information on the Configure Service page. Leave the other settings as default:

Launch Type: EC2
Task Definition: ecslab-blue-taskdef (Family) latest (Revision)
Service name: ecslab-blue-service
Number of tasks: 2

The Number of tasks determines the number of container instances the service launches and maintains. In this case you specify two tasks to launch two container instances. Even though for the sake of simplicity and to minimize costs this Lab uses a single EC2 host instance, you can alleviate the risk of having a single point of failure in the number of container instances you launch. The Minimum healthy percent and Maximum percent parameters help determine deployment strategies. The maximum parameter enables you to define the deployment batch size. With the value of 200%, the scheduler may start two new tasks before stopping the two older tasks. This can help you maintain consistent active capacity, but you may want to reduce the value to control costs if your environment is large and expensive to run. The minimum healthy parameter controls the lower limit of your service's tasks and is a way to maintain minimum capacity.

4. Click Next step. In the Configure network step enter the following values. Leave the other settings as default:

Load balancer type: Application Load Balancer
Service IAM role: ecsServiceRole
Load balancer name: ecslab-alb
Container name : port: ecslab-blue-container:0:8081

5. In the Container to load balance section, click Add to load balancer. In the fields that appear, select ecslab-targetgroup from the Target group name drop-down list. This automatically populates the other settings with the target group information you entered in an earlier Lab Step. Uncheck Enable service discovery integration checkbox. Click Next step

6. Click Configure Service Auto Scaling for Service Auto Scaling option. Though not used in this Lab, it is useful to examine the Auto Scaling options and understand how you can use Auto Scaling to maintain a target service level in your cluster. You can specify a minimum, maximum, and desired number of tasks

You can also set task scaling policies based on CloudWatch alarms. This allows your cluster to respond to increases or decreases in activity or traffic.

7. After exploring the scaling options, reset the Service Auto Scaling option to Do not adjust the service’s desired count to disable Auto Scaling. Click Next step

8. Review the service configuration on the Review step. Click Create Service.

9. After the Create Load Balancer and Create Service tasks complete, click View Service.

10. The ecslab-blue-service starts two tasks and begins the process of deploying them to the ECS cluster. Within approximately two minutes the service Status will change to ACTIVE and the Tasks tab will show two running tasks

11. Click the Events tab to view the service events

Note how the service registered two targets with the application load balancer target group. Earlier you created a load balancer but did not register any target instances. Instead, the service instantiates container instances based on the number of tasks you specify. You added the load balancer and target group to the service, so any new container instances the service creates automatically register as a target in the target group.

12. Return to Clusters > ecslab-cluster. Click the Create button on the Services tab. You are going to create the green service now, but you will not give it any tasks at this moment.

13. Enter the following information on the Create Service page. Leave the other settings as default.

Launch type: EC2
Task Definition: ecslab-green-taskdef (Family) latest (Revision)
Service name: ecslab-green-service
Number of tasks: 0

Note: You can create services with zero tasks. They will not launch any container instances upon creation. This allows you to prepare for future operations before your deployment is fully ready.

14. Click Next step. In the Configure network step, choose the following options:

Load balancer type: Application Load Balancer
Service IAM role: ecsServiceRole
Load balancer name: ecslab-alb
Container name : port: ecslab-green-container:0:8081
Uncheck Enable service discovery integration checkbox.

15. In the Container to load balance section, click Add to load balancer and select ecslab-targetgroup from the Target group name drop-down list. Click Next step.

16. Leave the Service Auto Scaling option set to Do not adjust the service’s desired count. Click Next step.

17. Click Create Service. After the Create Load Balancer and Create Service tasks complete, click View Service.

In this Lab Step, you created two services for your blue and green applications. You learned how these services control the desired capacity and how you can configure them to scale, if necessary. You also learned more about how they interact with the target group and load balancer you created earlier. You started two tasks for the blue application upon service creation. These tasks launched and registered two container instances.

Viewing Instances and Application Message
-----------------------------------------

Now that you have put all of the pieces in place to build and store Docker images, dynamically register container instances, and maintain a target capacity, it is time to learn more about the interdependent service actions and see some results. In this Lab Step you will view the resources launched by ECS and access the blue application launched when you created the blue service.

1. Search for EC2 in the service search bar to navigate to the EC2 Console.

2. Select INSTANCES > Instances from the EC2 Dashboard column. The ECS blue service automatically launched an EC2 instance to host the ECS container instances running your tasks

3. Select LOAD BALANCING > Target Groups from the EC2 Dashboard column. Click on the target group you created early.

4. Click the Targets tab and view the Registered targets section. Notice that you have two healthy targets despite having only one running EC2 instance. ECS deployed two container instances on the EC2 host instance. Each container instance dynamically registered with the target group on an assigned port in the ephemeral port range

5. Select LOAD BALANCING > Load Balancers from the EC2 Dashboard column.

6. Copy the DNS name from the Description tab and paste it into a browser window or tab. Append /api/ to the DNS value to form a Uniform Resource Locator (URL) that resembles this example

http://ecslab-alb-793477761.us-west-2.elb.amazonaws.com/api/

The load balancer DNS name provides a single point of access for your users/application for all instances behind the load balancer, even if those instances fail or are replaced.

7. Your browser will display the following message, though the format or appearance may vary slightly depending on the browser you use. 


Updating the Services and Application Deployments
-------------------------------------------------

1. Return to Services > EC2 Container Service > Amazon ECS > Clusters and click on the ecslab-cluster.

2. Select the ecslab-green-service on the Services tab and click Update

3. Change the Number of tasks value from 0 to 2. Click Next step through to the Review step and click Update Service.

4. Wait for the Service updated successfully message and click View Service.

5. Wait a few seconds for the new green tasks to launch. You can get service information on the service Events tab or wait for the Running count to reach 2.

6. Return to Services > Compute > EC2 > LOAD BALANCING > Target Groups and verify you have four healthy targets representing the four running container instances

The reduced resource requirements for a Docker container allow you to run multiple container instances on one EC2 instance. This works well for applications that require little resources, such as this simple message application.

7. Refresh your browser tab with the "Hello - I'm BLUE" message. It may take more than one refresh, but you will see the message alternate between "Hello - I'm BLUE" and "Hello - I'm GREEN" as you refresh the page and cycle through the container instances. Right now you have both applications running in their own pair of container instances. 

8. Return to Services > EC2 Container Service > Amazon ECS > Clusters and click on the ecslab-cluster.

9. Select the ecslab-blue-service on the Services tab and click Update.

10. Change the Number of tasks value from 2 to 0. Click Next step on each step until you can click Update Service.

11. Wait for the Service updated successfully message and click View Service.

12. Return to Services > Compute > EC2 > LOAD BALANCING > Target Groups. You can watch two of the container instance targets draining as ECS stops the tasks.

13. Refresh the browser tab that is pointing to the load balancer until the only message visible is "Hello - I'm GREEN". You have successfully transitioned from one application to another with no downtime or launching any new EC2 instances.

Note: There are different approaches to transitioning between one application or version to another. In this Lab Step, you bring up both applications and leave them running simultaneously and externally accessible for a short time. Another approach might be to schedule a maintenance window and set the number of running tasks for the blue application to 0. Wait for the blue targets to successfully drain. Then bring up the green application if you need to avoid overlap. Total downtime remains only minutes.

In this Lab Step, you launched the green application alongside the blue application in the ECS cluster. You watched as the round-robin load balancer distribution served messages from both applications running on one EC2 host instance. You then set the number of tasks for the blue application to 0 and watched as ECS drained the tasks and stopped serving the application's message.


Deleting the ECS Laboratory Resources
-------------------------------------

One of the great features of ECS is it helps make your application robust and available. If you manually terminate the EC2 instance running your ECS tasks, ECS starts another instance and relaunches the tasks. If you try to delete a service with running tasks, ECS will not allow it. ECS does an admirable job safeguarding running services and tasks inside a cluster. To stop an application you must either bring services down one at a time or delete the entire cluster.

1. Return to Services > EC2 Container Service > Amazon ECS > Clusters and click on the ecslab-cluster.

2. Click the Delete Cluster button in the upper-right.

3. Note the warning about deleting a CloudFormation stack. CloudFormation powers much of the ECS functionality and allows you to delete all of the resources in a cluster in an orderly fashion. Click Delete

4. Wait for the Delete Cluster processes to complete. After approximately one minute you will see the Deleted cluster ecslab-cluster successfully message.

5. Click Amazon ECR > Repositories in the left sidebar menu. Select the ca-container-registry repository and click Delete repository. Click Delete at the Delete repository warning indicating your images will also be deleted.

6. Return to Services > Compute > EC2 > LOAD BALANCING > Load Balancers from the EC2 Dashboard column. Select ecslab-alb and select Actions > Delete

7. Click Yes, Delete at the Delete Load Balancer message.

8. Select LOAD BALANCING > Target Groups from the EC2 Dashboard column. Select ecslab-targetgroup and select Actions > Delete

9. Click Yes at the Delete target group message.

In this Lab Step you deleted the key components of your ECS application including the cluster, image repository, target group, and load balancer. Other, ancillary resources remain such as the S3 source files, VPC, and CodeBuild projects. The Cloud Academy lab environment will delete these resources for you. In a real environment you may want to keep your source files and network infrastructure in case you want to update or redeploy an application later. These deletion steps stop the most expensive items from incurring additional charges. For many resource types, Amazon will charge a full hour if the resource is used during even one minute of that hour, so it is a good idea to delete resources when you are done using them.

References
----------

- 