<div align="center">
	<p>
		<img alt="Thoughtworks Logo" src="https://raw.githubusercontent.com/ThoughtWorks-DPS/static/master/thoughtworks_flamingo_wave.png?sanitize=true" width=200 />
    <br />
		<img alt="DPS Title" src="https://raw.githubusercontent.com/ThoughtWorks-DPS/static/master/EMPCPlatformStarterKitsImage.png" width=350/>
	</p>
  <h3>experiment-circleci-private-runners</h3>
</div>
<br />

A simple experiment to demonstrate using CircleCI Private Runners.  

### Dependencies

* [kind](https://kind.sigs.k8s.io)
* [kubectl](https://github.com/kubernetes/kubectl)
* [helm](https://github.com/helm/helm)
* [circleci cli](https://circleci.com/docs/local-cli/)

```bash
brew install kind kubectl helm circleci
```
Follow the instructions to authenticate to CircleCI using the CLI. If this is your first time using CircleCI you will need to [connect CircleCI to you github organization](https://circleci.com/docs/github-integration/).  

### Create CircleCI Organization namespace and resource class 

1. Create your namespace with the following command. `circleci namespace create <name> --org-id <your-organization-id>`  

An organization may only have a single namespace. The same namespace us used for both your organizations orbs as well as private runner resource classes.  

```bash
circleci namespace create twdps --org-id ThoughtWorks-DPS
```

2. Create a resource class for your self-hosted runner namespace with the following command. `circleci runner resource-class create <namespace>/<resource-class> <description> --generate-token`  

```bash
circleci runner resource-class create twdps/experiment 'private runner experiment' --generate-token

api:
    auth_token: f776a3e34**********
+------------------+---------------------------+
|  RESOURCE CLASS  |        DESCRIPTION        |
+------------------+---------------------------+
| twdps/experiment | private runner experiment |
+------------------+---------------------------+
```

### Define a circleci namespace in a local cluster

1. Create local Kubernetes instance using Kind.  

```bash
kind create cluster --name experiment

Creating cluster "experiment" ...
 ✓ Ensuring node image (kindest/node:v1.29.2) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-experiment"
You can now use your cluster with:

kubectl cluster-info --context kind-experiment
```

2. Create `circleci` namespace.  

```bash
kubectl create namespace circleci
```

### Install CircleCI container-agent

1. Add the container runner Helm repository by running the following command:
```bash
helm repo add container-agent https://packagecloud.io/circleci/container-agent/helm
helm repo update
```

2. Create a file called values.yaml file containing the following:
```yaml
agent:
  resourceClasses:
    <namespace>/<resource-class>:
      token: <resource_class_token>
```
Use your namespace, resource class, and runner token.  
```yaml
agent:
  resourceClasses:
    twdps/experiment:
      token: f776a3e34**********
```

3. Deploy the container agent.  
```bash
helm install container-agent container-agent/container-agent -n circleci -f values.yaml
```
If everything worked, you should see something like the following on your Self-Hosted Runners tab in CircleCI.  

<div align="center">
	<p>
		<img alt="example" src="https://raw.githubusercontent.com/ThoughtWorks-DPS/experiment-circleci-private-runners/main/sample-registered-container-agent.png" width=850/>
	</p>
</div>







See the [CircleCI documentation](https://circleci.com/docs/container-runner-installation/) for more detailed information including activating SSH login to runners.  