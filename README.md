# USACounts AWS Infrastructure

Stand up layers 1-5 of the Democracy application on AWS.

<kbd>
  <img src="./docs/private-task-public-loadbalancer.png" alt="infrastructure graphic"/>
</kbd>

## :rocket: Quick Start

1. Create the CodePipeline

    ```shell
    aws cloudformation create-stack \
    --stack-name Democracy-CodePipeline \
    --template-body file://aws/development/codepipeline.yml \
    --capabilities CAPABILITY_NAMED_IAM
    ```

2. Open the CodePipeline in the AWS console and **disable** the transition after the "Validate" stage

3. Trigger the CodePipeline by pushing to this repository

4. Create Base stack

    **All in one**

    ```shell
    aws cloudformation create-stack \
    --stack-name Democracy-Base-Stack \
    --template-body file://aws/base-stack.yml \
    --capabilities CAPABILITY_NAMED_IAM
    ```

    ---

    <details>
      <summary><b>
        Or Create the Base layers manually...
      </b></summary>

   1. Create the Network stack

       ```shell
       aws cloudformation create-stack \
       --stack-name Democracy-Network-Stack \
       --template-body file://aws/network-stack.yml \
       --capabilities CAPABILITY_NAMED_IAM
       ```

   2. Copy outputs of Network stack to `*-config.json` files and replace the default parameter values in `data-stack.yml`

   3. Create the Data stacks for each environment

       **Test**

       ```shell
       aws cloudformation create-stack \
       --stack-name Democracy-Test-Data-Stack \
       --template-body file://aws/data-stack.yml \
       --parameters ParameterKey=EnvironmentName,ParameterValue=test \
       --capabilities CAPABILITY_NAMED_IAM
       ```

       **UAT**

       ```shell
       aws cloudformation create-stack \
       --stack-name Democracy-UAT-Data-Stack \
       --template-body file://aws/data-stack.yml \
       --parameters ParameterKey=EnvironmentName,ParameterValue=uat \
       --capabilities CAPABILITY_NAMED_IAM
       ```

       **Prod**

       ```shell
       aws cloudformation create-stack \
       --stack-name Democracy-Prod-Data-Stack \
       --template-body file://aws/data-stack.yml \
       --parameters ParameterKey=EnvironmentName,ParameterValue=prod \
       --capabilities CAPABILITY_NAMED_IAM
       ```

  </details>

5. Copy outputs of Data stack to `*-config.json` files

6.  Open the CodePipeline in the AWS console and **enable** the transition after the "Validate" stage

7.  Trigger the CodePipeline by pushing to this repository

## :building_construction: The CodePipeline

The CodePipeline creates the stack located at `aws/main-stack.yml` across 3 stages (test, UAT, Prod)

1. Validates the Cloudformation files
2. Tests the stack deployment; then deletes
3. Create a change set with the UAT stack
4. Execute the change set of the UAT stack
5. Request manual approval
6. Create a change set with the Prod stack
7. Request manual approval
8. Execute the change set of the Prod stack

## :books: Resources

### Manually created

- CodePipeline
- S3 bucket "democracy-stack" for template storage
- Copy Linux AIM into us-east-1
- Create ECR repository for pushing scraper images
- Create ECR repository for pushing website images

### Automated by CodePipeline

- 3 Codebuild Projects
- VPC
- 2 Public subnets (across 2 Availabilty Zones)
- 2 Private subnets (across 2 Availabilty Zones)
- Internet Gateway (default route on the public subnets)
- NAT Gateways (one in each Availabilty Zone & default routes for in the private subnets)
- ECS cluster (AutoScaling Group)
- ECS hosts distributed across multiple Availability Zones
- 2 Task definitions / ECS services
- Lambda for ECS autoscaling
- RDS
- EFS
- Alias, CNAME, SSL Certificates

## :clipboard: TODO

- [x] Automate deployment of backend (EC2 for python migration)
- [x] Automate deployment of EFS bucket for bill migration data
- [ ] Automate deployment of Lambdas to invoke python migrations
- [x] Automate deployment of front end
- [x] Redirect traffic http -> https
- [x] Separate VPC, NAT Gateways, Internet Gateways from main-stack
- [x] Separate RDS from main-stack
- [x] Migrate launch type to Fargate
- [ ] Migrate Fargate cluster to [SPOT instances](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/fargate-capacity-providers.html)

## References

- [AWS infrastructure sample](https://github.com/awslabs/aws-cloudformation-templates/tree/master/aws/services/ECS)
