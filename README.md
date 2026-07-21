

to install built in openshift monitoring
 crc stop
crc config set enable-cluster-monitoring true
crc start
oc get pods -n openshift-monitoring
NAME                                                    READY   STATUS    RESTARTS   AGE
alertmanager-main-0                                     6/6     Running   6          14h
PS C:\Users\shrijandra> oc get pods -n openshift-user-workload-monitoring
No resources found in openshift-user-workload-monitoring namespace.
oc apply -f cluster_monitoring_config.yaml

oc get pods -n openshift-user-workload-monitoring
oc apply -f prometheus_rule.yaml
oc get prometheusrule
oc apply -f alertmanagerconfig.yaml
oc get alertmanagerconfig

oc adm policy add-scc-to-user anyuid -z nginx-sa
oc get alertmanager main -n openshift-monitoring -o yaml | Select-String "alertmanagerconfig"


oc -n openshift-monitoring patch alertmanager main --type merge -p '{\"spec\":{\"alertmanagerConfigSelector\":{\"matchLabels\":{}},\"alertmanagerConfigNamespaceSelector\":{\"matchLabels\":{}}}}'

create token for nginx-sa: oc create token nginx-sa -n demo
oc create secret generic eda-secret --from-literal=username=eda --from-literal=password=xxxxx -n demo

created in demo namespace
demo
├── nginx deployment
├── ServiceMonitor (not needed)
├── PrometheusRule
├── AlertmanagerConfig
└── eda-secret
oc create role deployment-scale-policy --verb=get,list,watch,patch,update --resource=deployments.apps --resource=deployments.apps/scale -n demo
oc create rolebinding deployment-scale-binding --role=deployment-scale-policy --serviceaccount=demo:nginx-sa -n demo  

oc scale deployment nginx --replicas=0 -n demo
oc port-forward -n openshift-monitoring pod/alertmanager-main-0 9093:9093
http://localhost:9093/#/alerts
