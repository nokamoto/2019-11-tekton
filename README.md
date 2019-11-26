# 2019-11-tekton

## GCP

### Installing Tekton Pipeline
https://github.com/tektoncd/pipeline/blob/master/docs/install.md

```bash
$ curl -sSL https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.6.0/release.yaml > release.v0.6.0.yaml
```

```bash
test_gcp_dks@cloudshell:~$ kubectl apply -f https://raw.githubusercontent.com/nokamoto/2019-11-tekton/master/release.v0.6.0.yaml
namespace/tekton-pipelines created
podsecuritypolicy.policy/tekton-pipelines created
clusterrole.rbac.authorization.k8s.io/tekton-pipelines-admin created
serviceaccount/tekton-pipelines-controller created
clusterrolebinding.rbac.authorization.k8s.io/tekton-pipelines-controller-admin created
customresourcedefinition.apiextensions.k8s.io/clustertasks.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/conditions.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/images.caching.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/pipelines.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/pipelineruns.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/pipelineresources.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/tasks.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/taskruns.tekton.dev created
service/tekton-pipelines-controller created
service/tekton-pipelines-webhook created
clusterrole.rbac.authorization.k8s.io/tekton-aggregate-edit created
clusterrole.rbac.authorization.k8s.io/tekton-aggregate-view created
configmap/config-artifact-bucket created
configmap/config-artifact-pvc created
configmap/config-defaults created
configmap/config-logging created
configmap/config-observability created
deployment.apps/tekton-pipelines-controller created
deployment.apps/tekton-pipelines-webhook created

test_gcp_dks@cloudshell:~$ kubectl get pods --namespace tekton-pipelines
NAME                                           READY   STATUS    RESTARTS   AGE
tekton-pipelines-controller-55c6b5b9f6-rptl9   1/1     Running   0          79s
tekton-pipelines-webhook-6794d5bcc8-2znch      1/1     Running   0          78s
```

### Hello World Tutorial /Task
https://github.com/tektoncd/pipeline/blob/master/docs/tutorial.md

```bash
test_gcp_dks@cloudshell:~$ # Get the tar.xz
test_gcp_dks@cloudshell:~$ curl -LO https://github.com/tektoncd/cli/releases/download/v0.5.0/tkn_0.5.0_Linux_x86_64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   620    0   620    0     0    672      0 --:--:-- --:--:-- --:--:--   672
100 10.5M  100 10.5M    0     0  3119k      0  0:00:03  0:00:03 --:--:-- 5633k
test_gcp_dks@cloudshell:~$ # Extract tkn to your PATH (e.g. /usr/local/bin)
test_gcp_dks@cloudshell:~$ sudo tar xvzf tkn_0.5.0_Linux_x86_64.tar.gz -C /usr/local/bin/ tkn
tkn
```

```bash
test_gcp_dks@cloudshell:~$ cat <<EOS | kubectl apply -f -
> apiVersion: tekton.dev/v1alpha1
> kind: Task
> metadata:
>   name: echo-hello-world
> spec:
>   steps:
>     - name: echo
>       image: ubuntu
>       command:
>         - echo
>       args:
>         - "hello world"
> EOS

test_gcp_dks@cloudshell:~$ cat <<EOS | kubectl apply -f -
> apiVersion: tekton.dev/v1alpha1
> kind: TaskRun
> metadata:
>   name: echo-hello-world-task-run
> spec:
>   taskRef:
>     name: echo-hello-world
> EOS
taskrun.tekton.dev/echo-hello-world-task-run created

test_gcp_dks@cloudshell:~$ tkn taskrun describe echo-hello-world-task-run
E1119 16:35:10.032610     611 metadata.go:241] Failed to unmarshal scopes: invalid character 'e' looking for beginning of value
Name:        echo-hello-world-task-run
Namespace:   default
Task Ref:    echo-hello-world

Status
STARTED          DURATION     STATUS
22 seconds ago   11 seconds   Succeeded

Input Resources
No resources

Output Resources
No resources

Params
No params

Steps
NAME
echo

test_gcp_dks@cloudshell:~$ tkn taskrun logs echo-hello-world-task-run
E1119 16:35:48.883505     620 metadata.go:241] Failed to unmarshal scopes: invalid character 'e' looking for beginning of value
[echo] hello world
```

#### Task Inputs and Outputs
```bash

$ kubectl create secret docker-registry regcred \
                    --docker-server=${DOCKER_SERVER} \
                    --docker-username=${DOCKER_USERNAME} \
                    --docker-password=${DOCKER_PASSWORD} \
                    --docker-email=${DOCKER_EMAIL}

$ cat <<EOS | kubectl apply -f -
apiVersion: tekton.dev/v1alpha1
kind: TaskRun
metadata:
  name: build-docker-image-from-git-source-task-run
spec:
  serviceAccountName: tutorial-service
  taskRef:
    name: build-docker-image-from-git-source
  inputs:
    resources:
      - name: docker-source
        resourceRef:
          name: skaffold-git
    params:
      - name: pathToDockerFile
        value: Dockerfile
      - name: pathToContext
        value: /workspace/docker-source/examples/microservices/leeroy-web #configure: may change according to your source
  outputs:
    resources:
      - name: builtImage
        resourceRef:
          name: skaffold-image-leeroy-web
EOS

$ kubectl get tekton-pipelines

$ tkn taskrun describe build-docker-image-from-git-source-task-run
```
