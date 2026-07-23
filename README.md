# OpenShift CRC Monitoring & Alerting Setup

## 1. Enable Built-in OpenShift Monitoring
Enable the built-in monitoring stack during CRC startup.

```bash
crc stop
crc config set enable-cluster-monitoring true
crc start
```

Verify monitoring pods:

```bash
oc get pods -n openshift-monitoring
```

---

## 2. Enable User Workload Monitoring
Deploy the configuration required for monitoring user applications.

Check current status:

```bash
oc get pods -n openshift-user-workload-monitoring
```

Apply configuration:

```bash
oc apply -f cluster_monitoring_config.yaml
```

Verify user workload monitoring pods:

```bash
oc get pods -n openshift-user-workload-monitoring
```

---

## 3. Create Prometheus Alert Rules
Deploy custom alert rules for your application.

```bash
oc apply -f prometheus_rule.yaml
```

Verify:

```bash
oc get prometheusrule
```

---

## 4. Configure Alertmanager
Create Alertmanager configuration to forward alerts.

```bash
oc apply -f alertmanagerconfig.yaml
```

Verify:

```bash
oc get alertmanagerconfig
```

---

## 5. Allow Alertmanager to Discover Configurations
Configure Alertmanager to load AlertmanagerConfig resources from user namespaces.

Check current configuration:

```bash
oc get alertmanager main -n openshift-monitoring -o yaml | Select-String "alertmanagerconfig"
```

Patch Alertmanager:

```bash
oc -n openshift-monitoring patch alertmanager main --type merge -p '{"spec":{"alertmanagerConfigSelector":{"matchLabels":{}},"alertmanagerConfigNamespaceSelector":{"matchLabels":{}}}}'
```

---

## 6. Create Service Account Token
Generate a token for the application service account.

```bash
oc create token nginx-sa -n demo
```

---

## 7. Create Secret for EDA Authentication
Create the secret used by Alertmanager to authenticate with Event-Driven Ansible.

```bash
oc create secret generic eda-secret \
  --from-literal=username=eda \
  --from-literal=password=<password> \
  -n demo
```

---

## 8. Demo Namespace Resources
Resources deployed in the `demo` namespace:

```
demo
├── nginx Deployment
├── PrometheusRule
├── AlertmanagerConfig
└── eda-secret
```

> **Note:** `ServiceMonitor` is optional and not required for this use case.

---

## 9. Grant Deployment Scaling Permissions
Allow the application service account to scale deployments.

Create Role:

```bash
oc create role deployment-scale-policy \
  --verb=get,list,watch,patch,update \
  --resource=deployments.apps \
  --resource=deployments.apps/scale \
  -n demo
```

Create RoleBinding:

```bash
oc create rolebinding deployment-scale-binding \
  --role=deployment-scale-policy \
  --serviceaccount=demo:nginx-sa \
  -n demo
```

Grant the required SCC:

```bash
oc adm policy add-scc-to-user anyuid -z nginx-sa
```

---

## 10. Start a Tunnel

```bash
ngrok http https://api.crc.testing:6443
```

---

## 11. Create a token
create token for nginx-sa: 
Get the API end point from the ngrok and use that endpoint to configure credential in AAP with below generated token. This credential will be used by AAP to execute the job template in CRC OpenShift cluster.

```bash
oc create token nginx-sa -n demo –duration=2h
```

---
## 12. Trigger an Alert
Scale the deployment to zero replicas to trigger the alert.

```bash
oc scale deployment nginx --replicas=0 -n demo
```

---

## 13. View Alertmanager Alerts
Port-forward Alertmanager and open the web UI.

```bash
oc port-forward -n openshift-monitoring pod/alertmanager-main-0 9093:9093
```

Open:

```
http://localhost:9093/#/alerts
```
