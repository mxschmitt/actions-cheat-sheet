# GitHub Actions Cheat sheet

Repository which contains often used patterns for automating the CI/CD workflow.

## Patterns

### Run a bash script

```yaml
- name: Deploying XYZ
  run: bash .github/workflows/deploy.sh
```

### Exit Bash script on errors

```bash
set -x
```

### Login to Docker GitHub Package Registry

```bash
echo "$DOCKER_PASSWORD" | docker login docker.pkg.github.com -u "$DOCKER_USERNAME" --password-stdin
```

### Build Docker Image

```bash
GIT_TAG="${GITHUB_REF##refs/tags/}"
GIT_BRANCH="${GITHUB_REF##refs/heads/}"

if [[ $GITHUB_REF != $GIT_TAG ]]; then
    # Use Git Tag as Docker Image Tag, so that it's connected to the corresponding GitHub release
    DOCKER_IMAGE_TAG="$GIT_TAG"
else
    DOCKER_IMAGE_TAG="$(echo $GIT_BRANCH | tr / -)"
fi

DOCKER_IMAGE_NAME="docker.pkg.github.com/$GITHUB_REPOSITORY/$(basename $GITHUB_REPOSITORY):$DOCKER_IMAGE_TAG"

docker build -f deploy/Dockerfile -t $DOCKER_IMAGE_NAME .
```

### Push Docker Image

```bash
docker push $DOCKER_IMAGE_NAME
```

### Install `kubectl` and `rancher`

```bash
sudo curl -sL -o /usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/v1.15.1/bin/linux/amd64/kubectl
sudo chmod +x /usr/local/bin/kubectl
sudo curl -sL https://github.com/rancher/cli/releases/download/v2.2.0/rancher-linux-amd64-v2.2.0.tar.gz | sudo tar xvz -C /usr/local/bin/ --strip-components=2
sudo chmod +x /usr/local/bin/rancher
```

### Run an Action on some branches and on `tags`

```yaml
if: |
    github.event_name == 'create' && github.event.ref_type == 'tag' ||
    (github.event_name == 'push' && (
    endsWith(github.event.ref, '/master') ||
    endsWith(github.event.ref, '/develop') ||
    contains(github.event.ref, '/deploy/')
    ))
```

### Update Rancher 2 Workload

```bash
rancher login --context $RANCHER_CONTEXT --token $RANCHER_TOKEN https://rancher.your-company.com/v3
rancher kubectl set image $RANCHER_RESOURCE $RANCHER_CONTAINER_NAME=$DOCKER_IMAGE_NAME --namespace $RANCHER_NAMESPACE
rancher kubectl rollout restart $RANCHER_RESOURCE --namespace $RANCHER_NAMESPACE
```
