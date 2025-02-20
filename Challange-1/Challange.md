# Challage 1 : Deploy the given architecture diagram for implementing a Jekyll SSG
***
### Diagram
![image](https://github.com/user-attachments/assets/5c86e442-02bf-408e-b392-3d54c556efda?width=250&height=400)
#### Step 1 : martin
1. Build user information for martin in the default kubeconfig file: User = martin , client-key = /root/martin.key and client-certificate = /root/martin.crt (Ensure don't embed within the kubeconfig file)
```
kubectl config set-credentials martin --client-certificate=/root/martin.crt --client-key=/root/martin.key 
```
2. Create a new context called 'developer' in the default kubeconfig file with 'user = martin' and 'cluster = kubernetes'
```
kubectl config set-context developer --user=martin --cluster=kubernetes
```
3. Check
```
kubectl config view
```
> 
controlplane ~ ➜  kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://controlplane:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: martin
  name: developer
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
- name: martin
  user:
    client-certificate: /root/martin.crt
    client-key: /root/martin.key


