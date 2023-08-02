# Pelorus demos

Collection of applications to test Pelorus with.

## basic-python

Contains a very simple Python HTTP server that just serves the `response.txt` file.

The file can easily be appended to, in order to demonstrate pipelines deploying changes.

### Requirements

- An OpenShift cluster with Pelorus installed
- A fork of this repository with access to commit and add webhooks
- A GitHub Personal Access Token (PAT) with access to read the forked repository issues

### Set up

Create a Pelorus instance with the following content, changing `<your_github_pat>` to your GitHub PAT and `<your_fork>` to your organization/user of the forked repository.
```yaml
apiVersion: charts.pelorus.dora-metrics.io/v1alpha1
kind: Pelorus
metadata:
  name: pelorus-basic-python-demo
spec:
  exporters:
    instances:
    - app_name: deploytime-exporter
      exporter_type: deploytime
      extraEnv:
      - name: NAMESPACES
        value: basic-python
    - app_name: committime-exporter
      exporter_type: committime
      extraEnv:
      - name: NAMESPACES
        value: basic-python
    - app_name: failure-exporter
      exporter_type: failure
      extraEnv:
      - name: PROVIDER
        value: github
      - name: TOKEN
        value: <your_github_pat>
      - name: PROJECTS
        value: <your_fork>/pelorus-demos
```

To create and start application, run
```sh
oc process -f basic-python.yaml -p=FORK_ORG=<your_fork> | oc apply -f -
```
changing `<your_fork>` to your organization/user of the forked repository.

Add webhook in the repository using this URL
```sh
oc describe bc/basic-python-app -n basic-python | \
grep 'webhooks/<secret>/github' | \
sed "s/<secret>/$(oc get bc/basic-python-app -o=jsonpath='{.spec.triggers..github.secret}' -n basic-python)/g"
```
[More details on configuring webhook](https://pelorus.readthedocs.io/en/latest/GettingStarted/QuickstartTutorial/#github-webhook).

After finishing testing, to stop and delete application, run
```sh
oc process -f basic-python.yaml -p=FORK_ORG=<your_fork> | oc delete -f -
```

### Test

To see Pelorus dashboards in action
- Create commits changing the `response.txt` file (because of the configured webhook, every commit will result in a deploy)
- Create and resolve issues with the `bug` and `app.kubernetes.io/name=basic-python-app` labels
