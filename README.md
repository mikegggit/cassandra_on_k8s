Cassandra on K8
===============
Demonstrates how to setup a cassandra cluster in kubernetes, and exercises the cluster through various use-cases.


Setup
-----
```
/tmp [chef:production] $ helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
"incubator" has been added to your repositories
```

```
/tmp [chef:production] $ helm fetch incubator/cassandra

/tmp [chef:production] $ ls cass*
cassandra-0.13.3.tgz

/tmp [chef:production] $ mv cassandra-0.13.3.tgz /usr/local/lib/helm/
/tmp [chef:production] $

/usr/local/lib/helm/ [chef:production] $ tar -xzvf cassandra-0.13.3.tgz
x cassandra/Chart.yaml
x cassandra/values.yaml
x cassandra/templates/NOTES.txt
x cassandra/templates/_helpers.tpl
x cassandra/templates/backup/cronjob.yaml
x cassandra/templates/backup/rbac.yaml
x cassandra/templates/configmap.yaml
x cassandra/templates/pdb.yaml
x cassandra/templates/service.yaml
x cassandra/templates/servicemonitor.yaml
x cassandra/templates/statefulset.yaml
x cassandra/.helmignore
x cassandra/README.md
x cassandra/sample/create-storage-gce.yaml

/usr/local/lib/helm/operators/cassandra [chef:production] $ helm template --set config.seed_size=2  -name cassandra --set config.seed_size=2  --namespace default --output-dir generated .
wrote generated/cassandra/templates/service.yaml
wrote generated/cassandra/templates/statefulset.yaml


/usr/local/lib/helm/operators/cassandra [chef:production] $ kubectl apply  --recursive --filename generated/
service/cassandra unchanged
statefulset.apps/cassandra configured
```

Verify
------
```
/usr/local/lib/helm/operators/cassandra [chef:production] $ kgp
NAME                      READY   STATUS    RESTARTS   AGE
cassandra-0               1/1     Running   0          23m
cassandra-1               1/1     Running   0          21m
cassandra-2               1/1     Running   0          19m
```

Issues
------
Problem: Unable to establish seed nodes in the cluster.
Detail:
Exception (org.apache.cassandra.exceptions.ConfigurationException) encountered during startup: The seed provider lists no seeds.
The seed provider lists no seeds.
ERROR [main] 2020-01-14 21:43:33,489 CassandraDaemon.java:708 - Exception encountered during startup: The seed provider lists no seeds.
Solution:
Check the statefulset yaml to verify seed list is correct.

A bad list might look like this...
```
        env:
        - name: CASSANDRA_SEEDS
          value: "cassandra-0.cassandra.ame.svc.cluster.local,cassandra-1.cassandra.ame.svc.cluster.local"
```

A good list might look like this...
```
        env:
        - name: CASSANDRA_SEEDS
          value: "cassandra-0.cassandra.default.svc.cluster.local,cassandra-1.cassandra.default.svc.cluster.local"
```

To fix the issue, manually specify the seed list according to the size of cluster.  To take advantage of the auto seed-list behaviors of the chart, pass in a namespace to replac the "ame" namespace that's being detected in the above bad example.

helm template --set config.seed_size=2  -name cassandra --set config.seed_size=2  --namespace default --output-dir generated .



Reference
---------
https://kubernetes.io/docs/tutorials/stateful-application/cassandra/
https://github.com/helm/charts/tree/master/incubator/cassandra

