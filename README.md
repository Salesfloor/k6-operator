 ![data flow](assets/data-flow.png)

# k6 Operator

`grafana/k6-operator` is a Kubernetes operator for running distributed [k6](https://github.com/grafana/k6) tests in your cluster.

This Salesfloor repository is a fork of the original [K6 Operator](https://github.com/grafana/k6-operator) repository. Please refer to the original repository for common info about K6 Operator.

## Salesfloor Features

- Runner jobs send 'test_starttime' tag as a part of real time output

## Salesfloor Specific Documentation

Steps to make a change available on SF Kubernetes:

1. make a change in the repository
2. build a new image
3. push the image to ghrc.io/salesfloor docker registry
4. install the updated K6 Operator to SF Kubernetes on Google Cloud

See below details on how to do it.

### Pre-requisites (Setup)

1. Install docker and make sure docker daemon is running
2. Install kubectl
3. Install gcloud
4. Authorize on google cloud

```bash
gcloud auth login
```

5. Connect to the 'Performance tests K6' project

```bash
gcloud container clusters get-credentials  \
       --project "perfomance-tests1-k6" \
       --region "northamerica-northeast1" \
      cluster-k6-tests
```

6. In your gitHub account (with access to Salesfloor organization) create [Personal Access Token (Classic)](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic) and select access permissions:
    - `read:packages`
    - `write:packages`
    - `delete:packages`

Note! please select the soonest available token expiration option. As long as K6 Operator may need updating and re-installing rarely, there is no need for a non-expiring token.

7. Set environment variables with the github token info

``` bash
    export GITHUB_USERNAME=<github-username> # the account where token is created
    export GITHUB_EMAIL=<github-email>
    export GITHUB_PACKAGES_TOKEN=<personal-access-token>
```

8. Login Docker to ghcr.io (to push images to ghcr.io/salesfloor/...)

```bash
make docker-login
```

9. Craete github-packages-secret secret on Kubernetes (so that Kubernetes can pull images from ghcr.io/salesfloor/...):

```bash
make kube-secret
```

### Make changes

Make sure to update unit tests for your changes. To execute unit tests, use these commands:

```bash
make test-setup # only need to run once
make test
```

### Build image

```bash
make docker-build
```

### Push image to Docker Registry

```bash
make docker-push
```

### Install K6 Operator to Kubernetes

```bash
make deploy
```

### Re-Install K6 Operator

To delete K6 Operator use `make delete` command.

Note! 

## Troubleshooting

- If `github-packages-secret` is absent or has wrong info `make deploy` command will still finish successfully. But Initializer job on Kubernetes will show Image Pull Error.

- `make delete` command also deletes `k6-operator-system` namespace together with related secrets. So make sure to run `make kube-secret` command again before using `make deploy` next time.

- Google Cloud Web interface is not quite transparent/clear in describing details of errors. `kubectl describe pod <pod-name>` command is much more helpful in troubleshooting.