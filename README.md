# Traderev Analytics

The Traderev analytics application consists of 4 components:

1. The static front-end (prod hostname: analytics.traderev.com, repo: https://github.com/TradeRev/tr-analytics)

2. The API backend (prod hostname: analytics-api.traderev.com, repo: https://github.com/TradeRev/tr-analytics-server-grails)

3. A reporting application that queries databases and sends reports out via email (repo: https://github.com/TradeRev/analytics-server)

4. A set of lambda functions that will POST to the reporting application (component #3) and a set of cloudwatch events that will call the lambda functions according to a schedule (repo: https://github.com/TradeRev/devops (tf-analytics terraform module))

Infrastructure and deployment Details
----------------------
1. The static front-end is hosted in an S3 bucket.  To deploy the content, clone the repository on a system with Node.js installed.  Then run cd into the repo directory and run 'npm install', then 'bower install'.  Edit gulpfile.js and ensure that the correct bucket names and endpoints are associated with each environment.  'gulp deploy --env production' will deploy to the production environment.

2. The API backend is run on an autoscaling group that is behind a load balancer.  The instances in the autoscaling group will need access to the analytics database and the datascience database.  The analytics database URL is specified in a YAML file named application.yml.  Presently, the datascience database URL is hard-coded into the application source.  A build process is run on the source code for the API backend which will result in a WAR file.  This WAR file is then uploaded to an S3 bucket that serves as an artifact repository.
Deployment of a selected WAR file consists of copying the WAR file into an S3 bucket designated for deployment.  When instances in the autoscaling group come up, they will pull down the WAR file from the deployment bucket and unpack it.  A copy of application.yml that is tailored to an environment will also be downloaded from S3.  

3. The reporting application is a Docker container that is run on ECS.  The build process consist of running 'docker build' and pushing the resulting image to ECR.  

4. The source code for the lambda functions are part of the 'devops' repo.  Deploying the Cloudwatch Events or the Lambda functions is done via Terraform.

Transitioning to Production
----------------------

- The reporting application is presently resides at analyticsserver.devops.traderev.com.  This is an public endpoint, but likely does not need to be.  In the new infrastructure setup built via Terraform, the load balancer is an internal load balancer and the lambda functions that call it will run inside a VPC.  It should be verified that there is nothing else that will require access to this endpoint.The load balancer is given an environment agnostic hostname, analyticsserver.traderev.internal.  Both the traderev.internal zone and the analyticsserver.traderev.internal hostname were manually created.  Terraform should eventually be used to create and maintain these resources.
- Presently, the code in tr-analytics-server-grails accesses the datascience database via a hard-coded URL that is compiled into a binary, and thus cannot be changed at runtime.  This will have to be changed before the application can be moved to production.  
- Presently, there are 19 lambda functions that post to analyticsserver.devops.traderev.com.  Most of these functions are close to identical.  The only significant difference between these functions is the 'postData' variable, which contains data this is part of the post that is sent to the server.  This 'postData' should be factored out of the functions and up to the Cloudwatch scheduled events that call the functions.  This way, only 1 lambda function needs to be maintained, and each of the Cloudwatch scheduled events can pass in the postData needed by the function.
