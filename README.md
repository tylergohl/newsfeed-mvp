# Newsfeed - MVP
<br>

## Getting Started

<br>

### **Prerequisites**

* AWS Account
* Docker: download can be found [here](https://www.docker.com/get-started)
* [ACT](https://github.com/nektos/act) - For running Github actions locally
    * Mac: (Assuming Homebrew is installed): `brew install act`
    * Win 10: Chocolatey currently not working, download the [latest release](https://github.com/nektos/act/releases/latest) and place the binary in your `PATH`.


### **Build / Deploy Environment**

* Make sure the following environment variables have been set:
    * `AWS_ACCESS_KEY_ID`
    * `AWS_SECRET_ACCESS_KEY`
    * `NEWSFEED_TOKEN` (this should be set to the value that the code expects for this secret)

* To build the environment, from the root directory of the repository run:

```
act -s AWS_ACCESS_KEY_ID -s AWS_SECRET_ACCESS_KEY -s NEWSFEED_TOKEN
```

This will build the environment in the default region of `us-west-2`.  If you would like to modify the region the environment is built in adjust the `AWS_DEFAULT_REGION` variable in both the `/.github/workflows/push.yaml` and `/.github/workflows/teardown.yaml`

**The script will output a line with the URL to connect to the newsfeed towards the bottom of the output**

<br>

### **Clean up / Tear down**

To clean up the environment and delete all resources when you are done, run:
```
act -s AWS_ACCESS_KEY_ID -s AWS_SECRET_ACCESS_KEY -s NEWSFEED_TOKEN -j teardown
```
<br>

### **Environment Details**

First - let's address the elephant in the room: Why aren't you building a k8s stack?!

Given the scope and constraints for this project - building a dedicated, self-managed k8s environment with all of its added complexity seemed like the wrong answer.  Building this on Fargate (or maybe even Lambda!) would have been my recommended approach but that would not meet the constraints provided.  [ECS Anywhere](https://aws.amazon.com/blogs/containers/introducing-amazon-ecs-anywhere/) might have been a good option that met the constraints but at the time of this writing it is unfortunately not yet generally available.

The Github Actions workflow for this repo does the following:

* Runs cfn-lint against the CloudFormation templates
* Checks for the existence of and adds a `secure-string` SSM Parameter for `newsfeed-token`
* Builds the following resources:
    * VPC
    * Two Application Load Balancers (one internal to the VPC and one external exposed to the public)
    * IAM EC2 Instance Role / Security Groups
    * 3 EC2 Autoscaling Groups / LB Target Groups
        * Quotes
        * Front-end
        * Newsfeed
* Hits the public endpoint of the site with `curl` until it receives a 200 response (or times out at 10 minutes)
* Provides the public endpoint URL of the site to the runner

<br>

# **Future Work**

### **Production improvements**

The templates in this repository can easily be extended to build a production ready platform. Since the templates already utilize a VPC and Application Load Balancer the following high level steps could be taken to build a production (or QA, Staging etc) environment:

* Add in parameter files / Systems Manager Parameter Store parameters for the environments we want to build
* Adjust values for those parameters accordingly for the production environment (e.g., `AsgMinSize` = 2, `InstanceType` = an instance type appropriate for the expected load)
* Extend the Github Actions `push.yaml` to build each environment that is desired
* If autoscaling is desired - configure the appropriate metrics to watch 
* If EC2 build times are too slow for autoscaling - consider utilizing Packer to build a pre-baked AMI that would eliminate most if not all of the `userdata` build time
* Add logging / monitoring / observability tooling
* Register a domain name, add a TLS certificate (using Certificate Manager for free!) to the external load balancer and enforce HTTPS.
    * If TLS is required for internal HTTP communication as well, AWS Private CA could be utilized

### **CI/CD considerations**

This environment is built using Github Actions.  Extending to prod from an infrastructure deployment perspective would just be a matter of adding in the appropriate environment files and sections in the `push.yaml` (Depending on the branching strategy desired) and adding some integration testing.

For an *initial* effort of a CI/CD pipeline from a development perspective I'd take the following approach:

* First - engage with the developers and business to understand their workflow.  What type of branching strategy are they utilizing?  What are the business requirements for uptime and reliability? How many environments and what types of environments are needed?  These types of conversations will help build the right initial pipeline.
* Assuming the development team wants their pushes to `master` to be deployed to prod, an initial pipeline might look like this:
    * Add in a Github Actions workflow on `push` to `master` to the development platform that builds, runs tests and then pushes the built code to s3 using a new folder name (s3 prefix) each time (could be based on the git ref, or whatever makes sense)
    * Add a step to the workflow that updates a Systems Manager Parameter Store parameter that indicates the location for the new code in s3
    * Modify the `ec2.yaml` in this repository to point to that Parameter Store reference so that it always knows where to get the latest code
    * Add a step to the workflow that initiates an [ASG Instance Refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) - this would trigger a rolling replacement of the instances for the service being deployed which would grab the new code on initialization
    * If a rollback was necessary, the SSM Parameter could be modified back to a previous value and another EC2 Instance Refresh could be executed.

This initial CI/CD effort could later be improved upon by implementing things like [Progressive Delivery](https://redmonk.com/jgovernor/2020/01/06/research-in-2020-stuff-i-am-thinking-about/), Canary Deploys etc.