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
3. Check output 
```
kubectl config view
```
```
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
```
***
### Step 2: jekyll-pvc
1. Storage Request: 1Gi
2. Access modes: ReadWriteMany
3. pvc name = jekyll-site, namespace = development
4. 'jekyll-site' PVC should be bound to the PersistentVolume called 'jekyll-site'.
```
vi pvc.yaml
```
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jekyll-site
  namespace: development
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  volumeName: jekyll-site
```
***
### Step 3: jekyll
1. pod: 'jekyll' has an initContainer, name: 'copy-jekyll-site', image: 'gcr.io/kodekloud/customimage/jekyll'
2. initContainer: 'copy-jekyll-site', command: [ "jekyll", "new", "/site" ] (command to run: jekyll new /site)
3. pod: 'jekyll', initContainer: 'copy-jekyll-site', mountPath = '/site'
4. pod: 'jekyll', initContainer: 'copy-jekyll-site', volume name = 'site'
5. pod: 'jekyll', container: 'jekyll', volume name = 'site'
6. pod: 'jekyll', container: 'jekyll', image = 'gcr.io/kodekloud/customimage/jekyll-serve'
7. pod: 'jekyll', uses volume called 'site' with pvc = 'jekyll-site'
8. pod: 'jekyll' uses label 'run=jekyll'
```
vi pod.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: jekyll
  name: jekyll
  namespace: development
spec:
  volumes:
    - name: site
      persistentVolumeClaim:
        claimName: jekyll-site
  initContainers:
  - name: copy-jekyll-site
    image: gcr.io/kodekloud/customimage/jekyll
    command: ['sh', '-c', 'jekyll', 'new', '/site' ]
	volumeMounts:
    - name: site
      mountPath: /site
  containers:
  - image: gcr.io/kodekloud/customimage/jekyll-serve
    name: jekyll
	volumeMounts:
    - name: site
      mountPath: /site
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```
***
### Step 4: develop-role
1. 'developer-role', should have all(*) permissions for services in development namespace
2. 'developer-role', should have all permissions(*) for persistentvolumeclaims in development namespace
3. 'developer-role', should have all(*) permissions for pods in development namespace
```
vi role.yaml
```
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: developer-role
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods", "persistentvolumeclaims" , "services"]
  verbs: ["*"]
```
***
### Step 5: developer-rolebinding
1. create rolebinding = developer-rolebinding, role= 'developer-role', namespace = development
2. rolebinding = developer-rolebinding associated with user = 'martin'
```
vi devrolebinding.yaml
```
```
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: developer-rolebinding
  namespace: development
subjects:
# You can specify more than one "subject"
- kind: User
  name: martin # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: developer-role  # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```
***
### Step 6: jelyll-node-service
1. Service 'jekyll' uses targetPort: '4000', namespace: 'development'
2. Service 'jekyll' uses Port: '8080', namespace: 'development'
3. Service 'jekyll' uses NodePort: '30097', namespace: 'development'
***
```
vi service.yaml
```
```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: jekyll
  name: jekyll
  namespace: development
spec:
  ports:
  - name: "4000"
    nodePort: 30097
    port: 8080
    protocol: TCP
    targetPort: 4000
  selector:
    app: jekyll
  type: NodePort
status:
  loadBalancer: {}
```

### Step 7: kube-config
1. set context 'developer' with user = 'martin' and cluster = 'kubernetes' as the current context.
```
kubectl config set-context developer --user=martin --cluster=kubernetes
```
```
kubectl config use-context developer
```



