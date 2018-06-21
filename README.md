# datadog-openshift
Some guidelines and learnings from OpenShift deployment of Datadog


To spin up an OpenShift instance using vagrant :

`cd os3.7-vagrant && vagrant up`

To use the cli client, run

```
./oc login https://192.168.33.22:8443 -u system -p admin
```

This user is has the cluster-admin role, so everything should work. If you need to run with the real admin account, you can run oc inside the origin docker container, with:

```
sudo docker exec -it origin bash
```

# Install
For each file in manifests, replace %%NAMESPACE%% with your project namespace, and replace %%API_KEY%% with your Datadog API key. Also be sure to put your tags into the `datadog-agent.yaml` file so they come into Datadog.

Create security constraints
```
./oc create -f manifests/datadog-securityConstraints.yaml
```

Create a service account
```
./oc create -f manifests/datadog-serviceAccount.yaml
```

Create a cluster role
```
./oc create -f manifests/datadog-clusterRole.yaml
```

Bind the cluster role to service account
```
./oc create -f manifests/datadog-clusterRoleBinding.yaml
```

Create a config map for adding custom configs to the agent
```
./oc create -f manifests/datadog-configMap.yaml
```

<!-- Ensure the security constraints are applied to the datadog user
```
./oc adm policy add-scc-to-user privileged -n %%NAMESPACE%% -z datadog
``` -->

Create the daemonset from manifest
```
./oc create -f manifests/datadog-agent.yaml
```

Check the pods are running as expected
```
./oc get pods
```

Output should look similar to the following
```console
NAME            READY     STATUS    RESTARTS   AGE
datadog-pgtm5   1/1       Running   0          11m
```

We can bash into the pod then to check the agent
```
./oc exec -it datadog-pgtm5 bash
```

You will see something like this, type in the commany "agent diagnose"
```console
I have no name!@datadog-pgtm5:/$ agent diagnose
```

The output should look similar to the following:
```console
=== Running ECS Fargate Metadata availability diagnosis ===
[ERROR] error: Get http://169.254.170.2/v2/metadata: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers) - 1529570640384116234
===> FAIL

=== Running Kubelet availability diagnosis ===
2018-06-21 08:44:00 UTC | ERROR | (diagnosis.go:33 in diagnoseFargate) | Get http://169.254.170.2/v2/metadata: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
[INFO] info: Skipping TLS verification - 1529570640390542592
2018-06-21 08:44:00 UTC | INFO | (init.go:34 in buildTLSConfig) | Skipping TLS verification
===> PASS

=== Running Kubernetes API Server availability diagnosis ===
===> PASS

=== Running EC2 Metadata availability diagnosis ===
[ERROR] error: unable to fetch EC2 API, Get http://169.254.169.254/latest/meta-data/hostname: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers) - 1529570640519242804
===> FAIL

=== Running GCE Metadata availability diagnosis ===
2018-06-21 08:44:00 UTC | ERROR | (diagnosis.go:21 in diagnose) | unable to fetch EC2 API, Get http://169.254.169.254/latest/meta-data/hostname: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
[ERROR] error: unable to retrieve hostname from GCE: Get http://169.254.169.254/computeMetadata/v1/instance/hostname: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers) - 1529570640820189512
===> FAIL

=== Running Azure Metadata availability diagnosis ===
2018-06-21 08:44:00 UTC | ERROR | (diagnosis.go:21 in diagnose) | unable to retrieve hostname from GCE: Get http://169.254.169.254/computeMetadata/v1/instance/hostname: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
[ERROR] error: Azure HostAliases: unable to query metadata endpoint: Get http://169.254.169.254/metadata/instance/compute/vmId?api-version=2017-04-02&format=text: dial tcp 169.254.169.254:80: connect: no route to host - 1529570640821679212
===> FAIL

=== Running Docker availability diagnosis ===
2018-06-21 08:44:00 UTC | ERROR | (diagnosis.go:21 in diagnose) | Azure HostAliases: unable to query metadata endpoint: Get http://169.254.169.254/metadata/instance/compute/vmId?api-version=2017-04-02&format=text: dial tcp 169.254.169.254:80: connect: no route to host
[INFO] info: successfully connected to docker - 1529570640823243308
2018-06-21 08:44:00 UTC | INFO | (diagnosis.go:26 in diagnose) | successfully connected to docker
[INFO] infof: successfully got hostname "oshift-ryan.nutley" from docker - 1529570640832659922
===> PASS

=== Running ECS Metadata availability diagnosis ===
2018-06-21 08:44:00 UTC | INFO | (diagnosis.go:33 in diagnose) | successfully got hostname "oshift-ryan.nutley" from docker
[ERROR] error: temporary failure in ecsutil, will retry later: Error: No such container: ecs-agent - 1529570640835827549
===> FAIL
```
