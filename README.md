# gitlab-runner

## Install gitlab-runner helm chart in the eks cluster

### Create new namespace in the cluster
```bash
kubectl create namespace gitlab-runner
```

### Set Context for the gitlab-runner namespace
```bash
kubectl config set-context --current --namespace=gitlab-runner
```

### Add the helm repository
```bash
helm repo add gitlab http://charts.gitlab.io/
```

### Create a values.yaml file for gitlab runner helm chart and update the values for <your gitlab url> and <your gitlab runner token>

```yaml
gitlabUrl: <your gitlab url>
rbac:
  clusterWideAccess: false
  create: true
  serviceAccountName: gitlab-runner
runnerToken: <your gitlab runner token>
runners:
  privileged: false
  serviceAccountName: gitlab-runner
  name: "gitlab-runner"
  executor: kubernetes
  config: |
    [[runners]]
      name = "runner-{{ .Release.Name }}"
      environment = ["HOME=/tmp", "builds_dir=/tmp"]
      url = "{{ .Values.gitlabUrl }}"
      executor = "kubernetes"
      builds_dir = "/tmp"
      [runners.kubernetes]
        privileged = false
        cpu_request = "500m"
        memory_request = "500Mi"
        service_cpu_limit = "1000m"
        service_memory_limit = "2000Mi"
        service_cpu_request = "150m"
        service_memory_request = "350Mi"
        helper_cpu_limit = "1500m"
        helper_memory_limit = "1500Mi"
        helper_cpu_request = "150m"
        helper_memory_request = "375Mi"
        poll_timeout = 600
        service_account = "gitlab-runner"
        [runners.kubernetes.node_tolerations]
          "cicd=true" = "NoSchedule"

```

### Install gitlab runner helm on EKS

```bash
helm install -f gitlab-runner.values.yaml gitlab-runner gitlab/gitlab-runner
```

### For testing now copy the example directory and .gitlab-ci.yml to your Gitlab project

### Then update the variables DOCKER_HUB_USERNAME, DOCKER_HUB_REPO on the .gitlab-ci.yml

### Add variable to the gitlab for DOCKER_HUB_PASSWORD




gitlab-runner register  --url https://gitlab.com  --token glrt-dhjaQoQyU3xsV-eYMkjk