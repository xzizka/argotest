### Testing scenarion
When Argo finishes the deployment, exec into pod:
```bash
kubectl exec -it nginx -n <namespace> -- /bin/bash
```
Then exec the following commands inside:
```bash
APISERVER=https://kubernetes.default.svc
SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount
NAMESPACE=$(cat ${SERVICEACCOUNT}/namespace)
TOKEN=$(cat ${SERVICEACCOUNT}/token)
CACERT=${SERVICEACCOUNT}/ca.crt
```
Now, let's test the access to various APIs
```bash
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/${NAMESPACE}/pods/
```
Role `testrole` can access list pods in the actual namespace. This role is granted to the `build-robot` service account used by the pod. You wll see the results.
```bash
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/${NAMESPACE}/secrets/
```
Access to secrets is not allowed in the current namespace to the current service account. You will see error `403`.
```bash
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/kube-system/secrets/
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/kube-system/pods/
```
No operation is allowed in the `kube-system` namespace. You will see error `403`.
