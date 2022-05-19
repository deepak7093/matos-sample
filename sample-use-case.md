# Cloudseals: EKS remediation

## Use case: 
EKS Insecure/http service ports

## Description: 
An existing cluster is communicating on port 443, now someone is trying to modify the port to 80 or 88 we have to generate a report of the log file as JSON and send it to the packet.

## Steps:
1. Get EKS ingress resource metadata and service port informations across all namespace within cluster
```python
from kubernetes import client, config
from pprint import pprint

conf = config.load_kube_config()
api_client = client.ApiClient(conf)
k8s_client = client.NetworkingV1Api(api_client)

try:
    ingress = k8s_client.list_ingress_for_all_namespaces()
    print(ingress.items)
except Exception as ex:
    print(ex)
```
2.Find the insecure http ports listening on other than 443

```python
for object in ingress.items:
    print(object.spec.rules)
    insecure_ports = []
    for rule in object.spec.rules:
        for path in rule.http.paths:
            insecure_ports.append(path.backend.service.portnumber)
            for rule in object.spec.rules:
        for path in rule.https.paths:
            if path.backend.service.port.number != 443:
                insecure_ports.append(path.backend.serviceport.number)
    print(insecure_ports)
```


3. Find the insecure http ports listening on other than 443
Perform test remediation using patch operation via REST with dry run
```python
    try:
        api_response = k8s_client.patch_namespaced_ingress(name, namespace, body, pretty=pretty, dry_run=dry_run)
        pprint(api_response)
    except ApiException as e:
        print("Exception when calling NetworkingV1Api->patch_namespaced_ingress: %s\n" % e)
```

4. Define remediation use case/best practices
    - All services must listen on secure and ssl ports i.e 443
    - Non https ports allowed
    - OPA policies must not allow ports other than 443

5. Integration with MetosSphere
