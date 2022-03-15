# Postgres for Kuberentes, the proper way.

PostgreSQL StatefulSet with replication.

# Prerequisites

Kubernetes cluster with Portworx installed.


# Create the Statefulset

```
camartinez@camartinez--MacBookPro16 postgres-sts % kubectl create namespace database

camartinez@camartinez--MacBookPro16 postgres-sts % kubectl apply -f postgres.yaml -n database

camartinez@camartinez--MacBookPro16 postgres-sts % kubectl apply -f pgpool.yaml -n database

camartinez@camartinez--MacBookPro16 postgres-sts % kubectl apply -f clientpod.yaml -n database

```


# Validate

You can connect to Postgres using the Kuberentes service

pgpool-svc.database.svc.cluster.local

or just

pgpool-svc

within the client pod.


```

camartinez@camartinez--MacBookPro16 postgres-sts % kubectl get all -n database
NAME                                    READY   STATUS              RESTARTS   AGE
pod/pg-client                           1/1     Running             0          20m
pod/pgpool-deployment-c4c84d565-mnfr7   1/1     Running             0          148m
pod/postgres-sts-0                      1/1     Running             0          5h40m
pod/postgres-sts-1                      1/1     Running             0          5h36m
pod/postgres-sts-2                      1/1     Running             0          5h36m

NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/pgpool-svc              ClusterIP   10.105.48.127    <none>        5432/TCP         5h40m
service/pgpool-svc-nodeport     NodePort    10.100.151.139   <none>        5432:32000/TCP   5h40m
service/postgres-headless-svc   ClusterIP   None             <none>        5432/TCP         5h40m

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/pgpool-deployment   1/1     1            1           5h40m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/pgpool-deployment-c4c84d565   1         1         1       5h40m

NAME                            READY   AGE
statefulset.apps/postgres-sts   3/3     5h40m


camartinez@camartinez--MacBookPro16 postgres-sts % kubectl get secret postgres-secrets -n database -o jsonpath="{.data.postgresql-password}" | base64 --decode


camartinez@camartinez--MacBookPro16 postgres-sts % kubectl exec -it pod/pg-client -n database -- /bin/bash

1001@pg-client:/$ PGPASSWORD=WbrTpN3g7q psql -h pgpool-svc.database.svc.cluster.local -p 5432 -U postgres
psql (11.12)
Type "help" for help.
postgres=# create database db1;
CREATE DATABASE
postgres=# \c db1;
postgres=# create table test (id int primary key not null, value text not null);
CREATE TABLE
postgres=# insert into test values (1, 'value1');
INSERT 0 1
postgres=# select * from test;
 id | value
----+--------
  1 | value1
(1 row)
postgres=# select * from pg_stat_replication;
 pid | usesysid | usename | application_name | client_addr | client_hostname | client_port |         backend_start         | backend_xmin |   state   | sent_lsn  | write_lsn | flush_lsn | replay_lsn |
 write_lag | flush_lag | replay_lag | sync_priority | sync_state
-----+----------+---------+------------------+-------------+-----------------+-------------+-------------------------------+--------------+-----------+-----------+-----------+-----------+------------+
-----------+-----------+------------+---------------+------------
 320 |    16384 | repmgr  | postgres-sts-1   | 10.244.4.64 |                 |       54784 | 2022-03-15 17:05:17.29478+00  |              | streaming | 0/964B360 | 0/964B360 | 0/964B360 | 0/964B360  |
           |           |            |             0 | async
 369 |    16384 | repmgr  | postgres-sts-2   | 10.244.4.65 |                 |       32800 | 2022-03-15 17:05:29.274765+00 |              | streaming | 0/964B360 | 0/964B360 | 0/964B360 | 0/964B360  |
           |           |            |             0 | async
(2 rows)
postgres=# \q
```