# IBM Cloud Code Engine

[IBM Code Engine](https://cloud.ibm.com/codeengine/overview) allows you to run your application, job or container on a managed serverless platform. Auto-scale workloads and only pay for the resources you consume.

IBM Code Engine exposes both Knative and Kubernetes Job Descriptor API. Lithops supports both of them. Follow IBM Code Engine documentation to get more details on the difference between those APIs.


## Configuration

1. If you don't have an IAM API key created, navigate to the [IBM IAM dashboard](https://cloud.ibm.com/iam/apikeys).

2. Click `Create an IBM Cloud API Key` and provide the necessary information.

3. Copy the generated IAM API key (You can only see the key the first time you create it, so make sure to copy it).

4. Naviagete to the [resource groups dashboard](https://cloud.ibm.com/account/resource-groups), and copy the desired resource group ID.

5. Edit your lithops config and add the following keys:

    ```yaml
    lithops:
        backend: code_engine

    ibm:
        iam_api_key: <IAM_API_KEY>
        region: <REGION>
        resource_group_id: <RESOURCE_GROUP_ID>
    ```

## Runtime

### Use your own runtime
If a pre-built runtime is not provided, Lithops will automatically build the default runtime the first time you run a script. For this task it uses the **docker** command installed locally in your machine. To make this working, you need:

1. [Install the Docker CE version](https://docs.docker.com/get-docker/).

2. Login to your container registry account:
   ```bash
   docker login
   ```

### Custom runtime

If you need to create a runtime with custom system packages and libraries, please follow [Building and managing Lithops runtimes to run the functions](https://github.com/lithops-cloud/lithops/tree/master/runtime/code_engine)


## Configure a private container registry for your runtime

### Configure Docker hub
To configure Lithops to access a private repository in your docker hub account, you need to extend the Code Engine config and add the following keys:

```yaml
code_engine:
    ....
    docker_server    : docker.io
    docker_user      : <container registry username>
    docker_password  : <container registry access TOEKN>
```

#### Configure IBM Container Registry
To configure Lithops to access to a private repository in your IBM Container Registry, you need to extend the Code Engine config and add the following keys:

```yaml
code_engine:
    ....
    docker_server    : us.icr.io  # Change-me if you have the CR in another region
    docker_user      : iamapikey
    docker_password  : <IBM IAM API KEY>
```


## Summary of configuration keys for IBM Cloud:

### IBM IAM:

|Group|Key|Default|Mandatory|Additional info|
|---|---|---|---|---|
|ibm | iam_api_key | |yes | IBM Cloud IAM API key to authenticate against IBM services. Obtain the key [here](https://cloud.ibm.com/iam/apikeys) |
|ibm | region | |yes | IBM Region.  One of: `eu-gb`, `eu-de`, `us-south`, `us-east`, `br-sao`, `ca-tor`, `jp-tok`, `jp-osa`, `au-syd` |
|ibm | resource_group_id | | yes | Resource group id from your IBM Cloud account. Get it from [here](https://cloud.ibm.com/account/resource-groups) |

## Code Engine:

|Group|Key|Default|Mandatory|Additional info|
|---|---|---|---|---|
|code_engine | project_name |  |no | Project name that already exists in Code Engine. If not provided lithops will automatically create a new project|
|code_engine | namespace |  |no | Alternatively to project_name, you can provide 'namespace'. Get it from you code engine k8s config file.|
|code_engine | region |  | no | Cluster region. One of: `eu-gb`, `eu-de`, `us-south`, `us-east`, `br-sao`, `ca-tor`, `jp-tok`, `jp-osa`, `au-syd`. Lithops will use the `region` set under the `ibm` section if it is not set here |
|code_engine | docker_server | docker.io |no | Docker server URL |
|code_engine | docker_user | |no | Docker hub username |
|code_engine | docker_password | |no | Login to your docker hub account and generate a new access token [here](https://hub.docker.com/settings/security)|
|code_engine | max_workers | 1000 | no | Max number of workers per `FunctionExecutor()`|
|code_engine | worker_processes | 1 | no | Number of Lithops processes within a given worker. This can be used to parallelize function activations within a worker. It is recommendable to set this value to the same number of CPUs of the container. |
|code_engine | runtime |  |no | Docker image name.|
|code_engine | runtime_cpu | 0.125 |no | CPU limit. Default 0.125vCPU. See [valid combinations](https://cloud.ibm.com/docs/codeengine?topic=codeengine-mem-cpu-combo) |
|code_engine | runtime_memory | 256 |no | Memory limit in MB. Default 256Mi. See [valid combinations](https://cloud.ibm.com/docs/codeengine?topic=codeengine-mem-cpu-combo) |
|code_engine | runtime_timeout | 600 |no | Runtime timeout in seconds. Default 600 seconds |
|code_engine | connection_retries | |no | If specified, number of job invoke retries in case of connection failure with error code 500 |


## Lithops using Knative API of Code Engine

The preferable way to run Lithops in Code Engine is by using the JOB API. However, Lithops can be also executed in Code Engine using the Knative API. To configure this mode of execution refer to the [Knative documentation](https://github.com/lithops-cloud/lithops/blob/master/config/compute/knative.md#configuration) and follow the steps to configure Knative.


## Test Lithops

Once you have your compute and storage backends configured, you can run a hello world function with:

```bash
lithops hello -b code_engine -s ibm_cos
```

## Viewing the execution logs

You can view the function executions logs in your local machine using the *lithops client*:

```bash
lithops logs poll
```

