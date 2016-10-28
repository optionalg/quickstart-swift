## Create Stack

### Prework
* Create an EC2 Keypair named `keypair_name` and save it to `keypair_path`

* Change directory to `swift-on-ecs` project

* Set up CloudFormation stack:

```
aws cloudformation create-stack \
  --stack-name swift-on-ecs \
  --capabilities CAPABILITY_IAM \
  --template-body file://demo/templates/swift-on-ecs.template \
  --parameters file://demo/templates/swift-on-ecs.config
 ```

**TODO**: Change from template-file to template-body once this is complete, and upload template to s3.

* SSH into the Dev Instance

```
{{SSHToBastion}}
```

**TIP**: Permissions for <keypair_path> are too open.

```
chmod 600 <keypair_path>
```

* Configure the AWS CLI with your account credentials

```
aws configure
```

* Run the Vapor project

```
cd ~/swift-on-ecs/demo/example && \
vapor run
```

* Validate that the server is running (in a new terminal window)

```
{{SSHToBastion}}
curl localhost:8080
```

* Validate that the cluster instance has been registered to the ECS cluster

* Pre-build a docker image to use in the demo
```
docker build \
  -t swift-on-ecs-prebuilt \
  --build-arg SWIFT_VERSION=DEVELOPMENT-SNAPSHOT-2016-06-06-a \
  --build-arg REPO_CLONE_URL=https://github.com/thomasphorton/swift-on-ecs.git \
  ~/swift-on-ecs/demo/example
```

## Demo

The demo begins on the generated development instance. A sample project is located at `~/swift-on-ecs/demo/example` and has already been built.

Once the local development is complete, the presenter will then walk through the process of building and tagging a Docker image. Once the image is tagged, it will be added to the ECS repository. At that point, the presentation will shift to setting up ECS, including task definitions, container definitions, and ultimately a cluster.

The final demo will be to access the ECS container instance and verify that the sample project is accessible from the internet.

### Deploy to ECS Using Docker

* Log in to ECS

```
aws ecr get-login
```

* Run log-in script by pasting results from previous command
* Build the docker image

**TODO**: Fix this! My personal GitHub isn't the best place for this- we need to figure out our strategy for managing assets (s3, codecommit, github).

```
docker build \
  -t swift-on-ecs \
  --build-arg SWIFT_VERSION=DEVELOPMENT-SNAPSHOT-2016-06-06-a \
  --build-arg REPO_CLONE_URL=https://github.com/thomasphorton/swift-on-ecs.git \
  ~/swift-on-ecs/demo/example
```

* While the build is running, open a new terminal window and continue.

```
{{SSHToBastion}}
```

* Tag the built image

```
{{TagPreBuiltImage}}
```

* Push the image to the ECS Repository

```
{{PushPreBuiltImage}}
```

* Create a Task Definition

### Amazon ECS -> Task Definitions

1. Task Definition Name: `swift-on-ecs-task`
2. Add Container
  * Container name: `swift-on-ecs-container`
  * Image: `{{RepositoryURL}}:latest`
  * Maximum memory: 300
  * Port mappings
    * Host port: 80
    * Container port: 8080
    * Protocol: tcp
  * Click `Add Container`
3. Click `Create`

* Create a Service

### Amazon ECS -> Clusters -> Services -> Create
1. Task Definition: `swift-on-ecs-task:1`
2. Service name: `swift-on-ecs-service`
3. Number of tasks: 1

### ELB Portion

### Configure Auto Scaling

* Connect to the Web Server
