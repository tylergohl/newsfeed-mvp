# Newsfeed - MVP


## Getting Started


### Prerequisites

* Docker: download can be found [here](https://www.docker.com/get-started)
* [ACT](https://github.com/nektos/act) - Run Github actions locally
    * Mac: (Assuming Homebrew is installed): `brew install act`
    * Win 10: (Assuming Chocolatey is installed): `choco install act-cli`

### Building

This will build the Dockerfile and create your execution environment.

```make build```

### Validating CloudFormation file

To validate the integrity of the CloudFormation file that will create the appropriate IAM roles

```make validate```

### Deployment

AWS CLI authentication needs to be configured

```make deploy```

### Debugging

If you need to debug a problem, you will need to exec into the Docker container. You can do that by simply typing in

```make exec```

