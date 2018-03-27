# 1. kuberenetes 설정 파일에 대한 환경 변수 등록(Kubeadm 툴 사용)
$ export KUBECONFIG=/etc/kubernetes/admin.conf
 
# 2. k8s RBAC 설정
kubectl create -f role.yaml
kubectl create -f role-binding.yaml
vi 00-default-admin-access.yaml
 
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: cluster-admin-clusterrolebinding-1
subjects:
- kind: ServiceAccount
  name: default
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: cluster-admin-clusterrolebinding-2
subjects:
- kind: ServiceAccount
  name: default
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
   
# 3. volume 설정
[root@localhost kubernetes]# kubectl create -f local-01.yaml
persistentvolume "pv0010" created
[root@localhost kubernetes]# cat local-01.yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv0010
  labels:
    type: local
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/tmp/data10"
 
[root@localhost kubernetes]# kubectl create -f local-02.yaml
persistentvolume "pv0011" created
[root@localhost kubernetes]# cat local-02.yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv0011
  labels:
    type: local
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/tmp/data11"
 
[root@localhost kubernetes]# kubectl get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM     STORAGECLASS   REASON    AGE
pv0010    5Gi        RWO            Retain           Available                                      10s
pv0011    5Gi        RWO            Retain           Available                                      8s
[root@localhost kubernetes]# kubectl get pvc
No resources found.
      
 
# 4. Initialize the cluster
./stolonctl --cluster-name=kube-stolon --store-backend=kubernetes --kube-resource-kind=configmap init
 
# 5. Create the sentinel
[root@localhost kubernetes]# kubectl create -f old_stolon-sentinel.yaml
[root@localhost kubernetes]# kubectl get pods --all-namespaces
kubectl logs -f stolon-sentinel-7955cd85f5-fhtk4
2018-03-19T02:13:06.973Z    DEBUG   cmd/sentinel.go:1746    cd dump: (*cluster.ClusterData)(0xc4201f7200)({
 FormatVersion: (uint64) 1,
 Cluster: (*cluster.Cluster)(0xc4203d2180)({
  UID: (string) (len=8) "584998c0",
  Generation: (int64) 1,
  ChangeTime: (time.Time) 2018-03-19 11:04:38.867299184 +0900 +0900,
  Spec: (*cluster.ClusterSpec)(0xc42016e870)({
   SleepInterval: (*cluster.Duration)(<nil>),
   RequestTimeout: (*cluster.Duration)(<nil>),
   ConvergenceTimeout: (*cluster.Duration)(<nil>),
   InitTimeout: (*cluster.Duration)(<nil>),
   SyncTimeout: (*cluster.Duration)(<nil>),
   FailInterval: (*cluster.Duration)(<nil>),
   DeadKeeperRemovalInterval: (*cluster.Duration)(<nil>),
   MaxStandbys: (*uint16)(<nil>),
   MaxStandbysPerSender: (*uint16)(<nil>),
   MaxStandbyLag: (*uint32)(<nil>),
   SynchronousReplication: (*bool)(<nil>),
   MinSynchronousStandbys: (*uint16)(<nil>),
   MaxSynchronousStandbys: (*uint16)(<nil>),
   AdditionalWalSenders: (*uint16)(<nil>),
   AdditionalMasterReplicationSlots: ([]string) <nil>,
   UsePgrewind: (*bool)(<nil>),
   InitMode: (*cluster.ClusterInitMode)(0xc4203089c0)((len=3) "new"),
   MergePgParameters: (*bool)(<nil>),
   Role: (*cluster.ClusterRole)(<nil>),
   NewConfig: (*cluster.NewConfig)(<nil>),
   PITRConfig: (*cluster.PITRConfig)(<nil>),
   ExistingConfig: (*cluster.ExistingConfig)(<nil>),
   StandbySettings: (*cluster.StandbySettings)(<nil>),
   DefaultSUReplAccessMode: (*cluster.SUReplAccessMode)(<nil>),
   PGParameters: (cluster.PGParameters) <nil>,
   PGHBA: ([]string) <nil>
  }),
  Status: (cluster.ClusterStatus) {
   CurrentGeneration: (int64) 0,
   Phase: (cluster.ClusterPhase) (len=12) "initializing",
   Master: (string) ""
  }
 }),
 Keepers: (cluster.Keepers) {
 },
 DBs: (cluster.DBs) {
 },
 Proxy: (*cluster.Proxy)(0xc4203d2240)({
  UID: (string) "",
  Generation: (int64) 0,
  ChangeTime: (time.Time) 0001-01-01 00:00:00 +0000 UTC,
  Spec: (cluster.ProxySpec) {
   MasterDBUID: (string) "",
   EnabledProxies: ([]string) <nil>
  },
  Status: (cluster.ProxyStatus) {
  }
 })
})
 
2018-03-19T02:13:06.974Z    DEBUG   cmd/sentinel.go:140 sentinelInfo dump   {"sentinelInfo": {"UID":"62319d9b"}}
2018-03-19T02:13:06.986Z    DEBUG   cmd/sentinel.go:1774    keepersInfo dump: (cluster.KeepersInfo) {
}
 
# 6.Create the keeper's password secret
[root@localhost kubernetes]# kubectl create -f old_secret.yaml
secret "stolon" created
 
#7. Create the stolon keepers statefulset
[root@localhost kubernetes]# kubectl create -f old_stolon-keeper.yaml
statefulset "stolon-keeper" created
[root@localhost kubernetes]# kubectl get pods --all-namespaces
NAMESPACE     NAME                                            READY     STATUS    RESTARTS   AGE
default       stolon-keeper-0                                 1/1       Running   0          8s
default       stolon-keeper-1                                 1/1       Running   0          4s
default       stolon-sentinel-6dd57b8b5f-kt45l                1/1       Running   0          1h
default       stolon-sentinel-6dd57b8b5f-tq2kz                1/1       Running   0          1h
kube-system   etcd-localhost.localdomain                      1/1       Running   0          4d
kube-system   kube-apiserver-localhost.localdomain            1/1       Running   0          4d
kube-system   kube-controller-manager-localhost.localdomain   1/1       Running   0          4d
kube-system   kube-dns-6f4fd4bdf-58bcr                        3/3       Running   0          4d
kube-system   kube-flannel-ds-rrw6g                           1/1       Running   0          4d
kube-system   kube-flannel-ds-tqtt6                           1/1       Running   0          3d
kube-system   kube-proxy-5hpwb                                1/1       Running   0          3d
kube-system   kube-proxy-k95zj                                1/1       Running   0          4d
kube-system   kube-scheduler-localhost.localdomain            1/1       Running   0          4d
kube-system   kubernetes-dashboard-5bd6f767c7-6rwfj           1/1       Running   0          3d
 
[root@localhost bin]# kubectl logs -f stolon-keeper-0
...
2018-03-19T08:18:44.749Z    DEBUG   cmd/keeper.go:1031  db status   {"initialized": true, "started": true}
2018-03-19T08:18:44.750Z    DEBUG   postgresql/postgresql.go:558    execing cmd {"cmd": {"Path":"/usr/lib/postgresql/9.6/bin/postgres","Args":["postgres","-V"],"Env":null,"Dir":"","Stdin":null,"Stdout":null,"Stderr":null,"ExtraFiles":null,"SysProcAttr":null,"Process":null,"ProcessState":null}}
2018-03-19T08:18:44.770Z    DEBUG   cmd/keeper.go:1418  target role {"targetRole": "master"}
2018-03-19T08:18:44.770Z    INFO    cmd/keeper.go:1423  our db requested role is master
2018-03-19T08:18:44.781Z    INFO    cmd/keeper.go:1451  already master
2018-03-19T08:18:44.786Z    DEBUG   postgresql/postgresql.go:558    execing cmd {"cmd": {"Path":"/usr/lib/postgresql/9.6/bin/postgres","Args":["postgres","-V"],"Env":null,"Dir":"","Stdin":null,"Stdout":null,"Stderr":null,"ExtraFiles":null,"SysProcAttr":null,"Process":null,"ProcessState":null}}
2018-03-19T08:18:44.799Z    INFO    cmd/keeper.go:1615  postgres parameters not changed
2018-03-19T08:18:44.799Z    INFO    cmd/keeper.go:1626  postgres hba entries not changed
2018-03-19T08:18:44.924Z    DEBUG   cmd/keeper.go:607   got configured pg parameters    {"pgParameters": {"datestyle":"iso, mdy","default_text_search_config":"pg_catalog.english","dynamic_shared_memory_type":"posix","hot_standby":"on","lc_messages":"en_US.utf8","lc_monetary":"en_US.utf8","lc_numeric":"en_US.utf8","lc_time":"en_US.utf8","listen_addresses":"127.0.0.1,10.244.1.24","log_timezone":"UTC","max_connections":"100","max_replication_slots":"20","max_wal_senders":"27","port":"5432","shared_buffers":"128MB","synchronous_standby_names":"","timezone":"UTC","unix_socket_directories":"/tmp","wal_keep_segments":"8","wal_level":"replica"}}
2018-03-19T08:18:44.924Z    DEBUG   cmd/keeper.go:614   filtered out managed pg parameters  {"filteredPGParameters": {"datestyle":"iso, mdy","default_text_search_config":"pg_catalog.english","dynamic_shared_memory_type":"posix","lc_messages":"en_US.utf8","lc_monetary":"en_US.utf8","lc_numeric":"en_US.utf8","lc_time":"en_US.utf8","log_timezone":"UTC","max_connections":"100","shared_buffers":"128MB","timezone":"UTC","wal_level":"replica"}}
2018-03-19T08:18:44.929Z    DEBUG   cmd/keeper.go:667   older wal file  {"filename": "000000010000000000000001"}
2018-03-19T08:18:47.453Z    DEBUG   cmd/keeper.go:607   got configured pg parameters    {"pgParameters": {"datestyle":"iso, mdy","default_text_search_config":"pg_catalog.english","dynamic_shared_memory_type":"posix","hot_standby":"on","lc_messages":"en_US.utf8","lc_monetary":"en_US.utf8","lc_numeric":"en_US.utf8","lc_time":"en_US.utf8","listen_addresses":"127.0.0.1,10.244.1.24","log_timezone":"UTC","max_connections":"100","max_replication_slots":"20","max_wal_senders":"27","port":"5432","shared_buffers":"128MB","synchronous_standby_names":"","timezone":"UTC","unix_socket_directories":"/tmp","wal_keep_segments":"8","wal_level":"replica"}}
2018-03-19T08:18:47.453Z    DEBUG   cmd/keeper.go:614   filtered out managed pg parameters  {"filteredPGParameters": {"datestyle":"iso, mdy","default_text_search_config":"pg_catalog.english","dynamic_shared_memory_type":"posix","lc_messages":"en_US.utf8","lc_monetary":"en_US.utf8","lc_numeric":"en_US.utf8","lc_time":"en_US.utf8","log_timezone":"UTC","max_connections":"100","shared_buffers":"128MB","timezone":"UTC","wal_level":"replica"}}
2018-03-19T08:18:47.457Z    DEBUG   cmd/keeper.go:667   older wal file  {"filename": "000000010000000000000001"}
2018-03-19T08:18:47.458Z    DEBUG   cmd/keeper.go:680   pgstate dump: (*cluster.PostgresState)(0xc4204ac630)({
 UID: (string) (len=8) "612f9867",
 Generation: (int64) 2,
 ListenAddress: (string) (len=11) "10.244.1.24",
 Port: (string) (len=4) "5432",
 Healthy: (bool) true,
 SystemID: (string) (len=19) "6534567330519445540",
 TimelineID: (uint64) 1,
 XLogPos: (uint64) 21949256,
 TimelinesHistory: (cluster.PostgresTimelinesHistory) {
 },
 PGParameters: (common.Parameters) (len=12) {
  (string) (len=9) "wal_level": (string) (len=7) "replica",
  (string) (len=12) "log_timezone": (string) (len=3) "UTC",
  (string) (len=7) "lc_time": (string) (len=10) "en_US.utf8",
  (string) (len=9) "datestyle": (string) (len=8) "iso, mdy",
  (string) (len=15) "max_connections": (string) (len=3) "100",
  (string) (len=14) "shared_buffers": (string) (len=5) "128MB",
  (string) (len=8) "timezone": (string) (len=3) "UTC",
  (string) (len=26) "default_text_search_config": (string) (len=18) "pg_catalog.english",
  (string) (len=10) "lc_numeric": (string) (len=10) "en_US.utf8",
  (string) (len=26) "dynamic_shared_memory_type": (string) (len=5) "posix",
  (string) (len=11) "lc_monetary": (string) (len=10) "en_US.utf8",
  (string) (len=11) "lc_messages": (string) (len=10) "en_US.utf8"
 },
 SynchronousStandbys: ([]string) {
 },
 OlderWalFile: (string) (len=24) "000000010000000000000001"
})
 
WARNING:  skipping special file "./postgresql.auto.conf"
 
[root@localhost kubernetes]# kubectl get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                          STORAGECLASS   REASON    AGE
pv0010    5Gi        RWO            Retain           Bound     default/data-stolon-keeper-1                            1m
pv0011    5Gi        RWO            Retain           Bound     default/data-stolon-keeper-0                            1m
[root@localhost kubernetes]# kubectl get pvc
NAME                   STATUS    VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-stolon-keeper-0   Bound     pv0011    5Gi        RWO                           47s
data-stolon-keeper-1   Bound     pv0010    5Gi        RWO
 
 
# 8.Create the proxies
[root@localhost kubernetes]# kubectl create -f old_stolon-proxy.yaml
deployment "stolon-proxy" created
[root@localhost kubernetes]# kubectl get pods --all-namespaces
NAMESPACE     NAME                                            READY     STATUS    RESTARTS   AGE
default       stolon-keeper-0                                 1/1       Running   0          6m
default       stolon-keeper-1                                 1/1       Running   0          6m
default       stolon-proxy-69c5cf4c76-4f9gj                   0/1       Running   0          14s
default       stolon-proxy-69c5cf4c76-gfzrf                   0/1       Running   0          14s
default       stolon-sentinel-6dd57b8b5f-kt45l                1/1       Running   0          1h
default       stolon-sentinel-6dd57b8b5f-tq2kz                1/1       Running   0          1h
kube-system   etcd-localhost.localdomain                      1/1       Running   0          4d
kube-system   kube-apiserver-localhost.localdomain            1/1       Running   0          4d
kube-system   kube-controller-manager-localhost.localdomain   1/1       Running   0          4d
kube-system   kube-dns-6f4fd4bdf-58bcr                        3/3       Running   0          4d
kube-system   kube-flannel-ds-rrw6g                           1/1       Running   0          4d
kube-system   kube-flannel-ds-tqtt6                           1/1       Running   0          3d
kube-system   kube-proxy-5hpwb                                1/1       Running   0          3d
kube-system   kube-proxy-k95zj                                1/1       Running   0          4d
kube-system   kube-scheduler-localhost.localdomain            1/1       Running   0          4d
kube-system   kubernetes-dashboard-5bd6f767c7-6rwfj           1/1       Running   0          3d
 
 
# 9.Create the proxy service
[root@localhost kubernetes]# kubectl create -f old_stolon-proxy-service.yaml
service "stolon-proxy-service" created
[root@localhost kubernetes]# kubectl get pods --all-namespaces
NAMESPACE     NAME                                            READY     STATUS    RESTARTS   AGE
default       stolon-keeper-0                                 1/1       Running   0          7m
default       stolon-keeper-1                                 1/1       Running   0          6m
default       stolon-proxy-69c5cf4c76-4f9gj                   1/1       Running   0          44s
default       stolon-proxy-69c5cf4c76-gfzrf                   1/1       Running   0          44s
default       stolon-sentinel-6dd57b8b5f-kt45l                1/1       Running   0          1h
default       stolon-sentinel-6dd57b8b5f-tq2kz                1/1       Running   0          1h
kube-system   etcd-localhost.localdomain                      1/1       Running   0          4d
kube-system   kube-apiserver-localhost.localdomain            1/1       Running   0          4d
kube-system   kube-controller-manager-localhost.localdomain   1/1       Running   0          4d
kube-system   kube-dns-6f4fd4bdf-58bcr                        3/3       Running   0          4d
kube-system   kube-flannel-ds-rrw6g                           1/1       Running   0          4d
kube-system   kube-flannel-ds-tqtt6                           1/1       Running   0          3d
kube-system   kube-proxy-5hpwb                                1/1       Running   0          3d
kube-system   kube-proxy-k95zj                                1/1       Running   0          4d
kube-system   kube-scheduler-localhost.localdomain            1/1       Running   0          4d
 
# 10. Connect to the db
[root@localhost kubernetes]# kubectl get svc
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes             ClusterIP   10.96.0.1        <none>        443/TCP    4d
stolon-proxy-service   ClusterIP   10.103.229.134   <none>        5432/TCP   21s
 
 
# 11.Connect to the proxy service
[root@localhost kubernetes]# psql --host 10.103.229.134  --port 5432 postgres -U stolon -W
Password for user stolon: password1
psql (9.6.6, server 9.6.8)
Type "help" for help.
 
postgres=# select version();
                                                          version                                                         
---------------------------------------------------------------------------------------------------------------------------
 PostgreSQL 9.6.8 on x86_64-pc-linux-gnu (Debian 9.6.8-1.pgdg80+1), compiled by gcc (Debian 4.9.2-10+deb8u1) 4.9.2, 64-bit
(1 row)
 
postgres=#
 
 
# 12.Create a test table and insert a row
postgres=# create table test (id int primary key not null, value text not null);
CREATE TABLE
postgres=# insert into test values (1, 'value1');
INSERT 0 1
postgres=# select * from test;
 id | value 
----+--------
  1 | value1
(1 row)
 
postgres=#
 
 
 
# 구성확인
[root@localhost kubernetes]# /home/bylee/dev/stolon-test/stolon/bin/stolonctl --cluster-name=kube-stolon --store-backend=kubernetes --kube-resource-kind=configmap status
=== Active sentinels ===
 
ID      LEADER
4410c9bb    false
5ec7a064    true
 
=== Active proxies ===
 
ID
d6777028
d989346e
 
=== Keepers ===
 
UID HEALTHY PG LISTENADDRESS    PG HEALTHY  PG WANTEDGENERATION PG CURRENTGENERATION
keeper0 true    172.17.0.7:5432     true        3           3  
keeper1 true    172.17.0.8:5432     true        2           2  
 
=== Cluster Info ===
 
Master: keeper0
 
===== Keepers/DB tree =====
 
keeper0 (master)
└─keeper1
