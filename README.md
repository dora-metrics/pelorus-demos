# Pelorus demos

Collection of applications to test Pelorus with.

## basic-python

Contains a very simple Python HTTP server that just serves the `response.txt` file.

The file can easily be appended to, in order to demonstrate pipelines deploying changes.

<!-- TODO To run this demo, fork it to your organization/user -->

Create a Pelorus instance with the following content
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
        value: <your_github_api_token>
      - name: PROJECTS
        value: dora-metrics/pelorus-demos
```

To create and start application, run
```sh
oc process -f basic-python.yaml | oc apply -f -
```

Add webhook in the repository using this URL
```sh
oc describe bc/basic-python-app -n basic-python | \
grep 'webhooks/<secret>/github' | \
sed "s/<secret>/$(oc get bc/basic-python-app -o=jsonpath='{.spec.triggers..github.secret}' -n basic-python)/g"
```
TODO more details https://pelorus.readthedocs.io/en/latest/GettingStarted/QuickstartTutorial/#github-webhook

Now, create commits (changing `response.txt` file), issues (with the `bug` and `app.kubernetes.io/name=basic-python-app` labels) and resolve issues in the repository to see Pelorus dashboards in action.

To stop and delete application, run
```sh
oc process -f basic-python.yaml | oc delete -f -
```
