# 기본 명령어
kubectl 의 기본적인 사용법을 익힌다. 

kubectl의 역할은 쿠버네티스이 상태를 확인하고 원하는 상태를 요청하는 것이다.

### kubectl 명령어

#### 상태설정하기(apply)
```shell
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl apply -h
Apply a configuration to a resource by file name or stdin. The resource name must be specified. This resource will be
created if it doesn't exist yet. To use 'apply', always create the resource initially with either 'apply' or 'create
--save-config'.

 JSON and YAML formats are accepted.

 Alpha Disclaimer: the --prune functionality is not yet complete. Do not use unless you are aware of what the current
state is. See https://issues.k8s.io/34274.

Examples:
  # Apply the configuration in pod.json to a pod
  kubectl apply -f ./pod.json

  # Apply resources from a directory containing kustomization.yaml - e.g. dir/kustomization.yaml
  kubectl apply -k dir/

  # Apply the JSON passed into stdin to a pod
  cat pod.json | kubectl apply -f -

  # Apply the configuration from all files that end with '.json'
  kubectl apply -f '*.json'

  # Note: --prune is still in Alpha
  # Apply the configuration in manifest.yaml that matches label app=nginx and delete all other resources that are not in
the file and match label app=nginx
  kubectl apply --prune -f manifest.yaml -l app=nginx

  # Apply the configuration in manifest.yaml and delete all the other config maps that are not in the file
  kubectl apply --prune -f manifest.yaml --all --prune-allowlist=core/v1/ConfigMap

Available Commands:
  edit-last-applied   Edit latest last-applied-configuration annotations of a resource/object
  set-last-applied    Set the last-applied-configuration annotation on a live object to match the contents of a file
  view-last-applied   View the latest last-applied-configuration annotations of a resource/object

Options:
    --all=false:
        Select all resources in the namespace of the specified resource types.

    --allow-missing-template-keys=true:
        If true, ignore any errors in templates when a field or map key is missing in the template. Only applies to
        golang and jsonpath output formats.

    --cascade='background':
        Must be "background", "orphan", or "foreground". Selects the deletion cascading strategy for the dependents
        (e.g. Pods created by a ReplicationController). Defaults to background.

    --dry-run='none':
        Must be "none", "server", or "client". If client strategy, only print the object that would be sent, without
        sending it. If server strategy, submit server-side request without persisting the resource.

    --field-manager='kubectl-client-side-apply':
        Name of the manager used to track field ownership.

    -f, --filename=[]:
        The files that contain the configurations to apply.

    --force=false:
        If true, immediately remove resources from API and bypass graceful deletion. Note that immediate deletion of
        some resources may result in inconsistency or data loss and requires confirmation.

    --force-conflicts=false:
        If true, server-side apply will force the changes against conflicts.

    --grace-period=-1:
        Period of time in seconds given to the resource to terminate gracefully. Ignored if negative. Set to 1 for
        immediate shutdown. Can only be set to 0 when --force is true (force deletion).

    -k, --kustomize='':
        Process a kustomization directory. This flag can't be used together with -f or -R.

    --openapi-patch=true:
        If true, use openapi to calculate diff when the openapi presents and the resource can be found in the openapi
        spec. Otherwise, fall back to use baked-in types.

    -o, --output='':
        Output format. One of: (json, yaml, name, go-template, go-template-file, template, templatefile, jsonpath,
        jsonpath-as-json, jsonpath-file).

    --overwrite=true:
        Automatically resolve conflicts between the modified and live configuration by using values from the modified
        configuration

    --prune=false:
        Automatically delete resource objects, that do not appear in the configs and are created by either apply or
        create --save-config. Should be used with either -l or --all.

    --prune-allowlist=[]:
        Overwrite the default allowlist with <group/version/kind> for --prune

    -R, --recursive=false:
        Process the directory used in -f, --filename recursively. Useful when you want to manage related manifests
        organized within the same directory.

    -l, --selector='':
        Selector (label query) to filter on, supports '=', '==', and '!='.(e.g. -l key1=value1,key2=value2). Matching
        objects must satisfy all of the specified label constraints.

    --server-side=false:
        If true, apply runs in the server instead of the client.

    --show-managed-fields=false:
        If true, keep the managedFields when printing objects in JSON or YAML format.

    --template='':
        Template string or path to template file to use when -o=go-template, -o=go-template-file. The template format
        is golang templates [http://golang.org/pkg/text/template/#pkg-overview].

    --timeout=0s:
        The length of time to wait before giving up on a delete, zero means determine a timeout from the size of the
        object

    --validate='strict':
        Must be one of: strict (or true), warn, ignore (or false).              "true" or "strict" will use a schema to validate
        the input and fail the request if invalid. It will perform server side validation if ServerSideFieldValidation
        is enabled on the api-server, but will fall back to less reliable client-side validation if not.               "warn" will
        warn about unknown or duplicate fields without blocking the request if server-side field validation is enabled
        on the API server, and behave as "ignore" otherwise.            "false" or "ignore" will not perform any schema
        validation, silently dropping any unknown or duplicate fields.

    --wait=false:
        If true, wait for resources to be gone before returning. This waits for finalizers.

Usage:
  kubectl apply (-f FILENAME | -k DIRECTORY) [options]

Use "kubectl apply <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```
원하는 리소스의 상태를 YAML파일로 작성하고 apply 명령어로 실행한다.
```shell
# 다시 한번 워드프레스 배포하기 (URL로!)
kubectl apply -f https://subicura.com/k8s/code/guide/index/wordpress-k8s.yml


roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl apply -f https://subicura.com/k8s/code/guide/index/wordpress-k8s.yml
deployment.apps/wordpress-mysql created
service/wordpress-mysql created
deployment.apps/wordpress created
service/wordpress created
```
#### 리소스 목록보기(get)
```shell
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl get -h
Display one or many resources.

 Prints a table of the most important information about the specified resources. You can filter the list using a label
selector and the --selector flag. If the desired resource type is namespaced you will only see results in your current
namespace unless you pass --all-namespaces.

 By specifying the output as 'template' and providing a Go template as the value of the --template flag, you can filter
the attributes of the fetched resources.

Use "kubectl api-resources" for a complete list of supported resources.

Examples:
  # List all pods in ps output format
  kubectl get pods

  # List all pods in ps output format with more information (such as node name)
  kubectl get pods -o wide

  # List a single replication controller with specified NAME in ps output format
  kubectl get replicationcontroller web

  # List deployments in JSON output format, in the "v1" version of the "apps" API group
  kubectl get deployments.v1.apps -o json

  # List a single pod in JSON output format
  kubectl get -o json pod web-pod-13je7

  # List a pod identified by type and name specified in "pod.yaml" in JSON output format
  kubectl get -f pod.yaml -o json

  # List resources from a directory with kustomization.yaml - e.g. dir/kustomization.yaml
  kubectl get -k dir/

  # Return only the phase value of the specified pod
  kubectl get -o template pod/web-pod-13je7 --template={{.status.phase}}

  # List resource information in custom columns
  kubectl get pod test-pod -o custom-columns=CONTAINER:.spec.containers[0].name,IMAGE:.spec.containers[0].image

  # List all replication controllers and services together in ps output format
  kubectl get rc,services

  # List one or more resources by their type and names
  kubectl get rc/web service/frontend pods/web-pod-13je7

  # List the 'status' subresource for a single pod
  kubectl get pod web-pod-13je7 --subresource status

Options:
    -A, --all-namespaces=false:
        If present, list the requested object(s) across all namespaces. Namespace in current context is ignored even
        if specified with --namespace.

    --allow-missing-template-keys=true:
        If true, ignore any errors in templates when a field or map key is missing in the template. Only applies to
        golang and jsonpath output formats.

    --chunk-size=500:
        Return large lists in chunks rather than all at once. Pass 0 to disable. This flag is beta and may change in
        the future.

    --field-selector='':
        Selector (field query) to filter on, supports '=', '==', and '!='.(e.g. --field-selector
        key1=value1,key2=value2). The server only supports a limited number of field queries per type.

    -f, --filename=[]:
        Filename, directory, or URL to files identifying the resource to get from a server.

    --ignore-not-found=false:
        If the requested object does not exist the command will return exit code 0.

    -k, --kustomize='':
        Process the kustomization directory. This flag can't be used together with -f or -R.

    -L, --label-columns=[]:
        Accepts a comma separated list of labels that are going to be presented as columns. Names are case-sensitive.
        You can also use multiple flag options like -L label1 -L label2...

    --no-headers=false:
        When using the default or custom-column output format, don't print headers (default print headers).

    -o, --output='':
        Output format. One of: (json, yaml, name, go-template, go-template-file, template, templatefile, jsonpath,
        jsonpath-as-json, jsonpath-file, custom-columns, custom-columns-file, wide). See custom columns
        [https://kubernetes.io/docs/reference/kubectl/#custom-columns], golang template
        [http://golang.org/pkg/text/template/#pkg-overview] and jsonpath template
        [https://kubernetes.io/docs/reference/kubectl/jsonpath/].

    --output-watch-events=false:
        Output watch event objects when --watch or --watch-only is used. Existing objects are output as initial ADDED
        events.

    --raw='':
        Raw URI to request from the server.  Uses the transport specified by the kubeconfig file.

    -R, --recursive=false:
        Process the directory used in -f, --filename recursively. Useful when you want to manage related manifests
        organized within the same directory.

    -l, --selector='':
        Selector (label query) to filter on, supports '=', '==', and '!='.(e.g. -l key1=value1,key2=value2). Matching
        objects must satisfy all of the specified label constraints.

    --server-print=true:
        If true, have the server return the appropriate table output. Supports extension APIs and CRDs.

    --show-kind=false:
        If present, list the resource type for the requested object(s).

    --show-labels=false:
        When printing, show all labels as the last column (default hide labels column)

    --show-managed-fields=false:
        If true, keep the managedFields when printing objects in JSON or YAML format.

    --sort-by='':
        If non-empty, sort list types using this field specification.  The field specification is expressed as a
        JSONPath expression (e.g. '{.metadata.name}'). The field in the API resource specified by this JSONPath
        expression must be an integer or a string.

    --subresource='':
        If specified, gets the subresource of the requested object. Must be one of [status scale]. This flag is beta
        and may change in the future.

    --template='':
        Template string or path to template file to use when -o=go-template, -o=go-template-file. The template format
        is golang templates [http://golang.org/pkg/text/template/#pkg-overview].

    -w, --watch=false:
        After listing/getting the requested object, watch for changes.

    --watch-only=false:
        Watch for changes to the requested object(s), without listing/getting first.

Usage:
  kubectl get
[(-o|--output=)json|yaml|name|go-template|go-template-file|template|templatefile|jsonpath|jsonpath-as-json|jsonpath-file|custom-columns|custom-columns-file|wide]
(TYPE[.VERSION][.GROUP] [NAME | -l label] | TYPE[.VERSION][.GROUP]/NAME ...) [flags] [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```
pod 조회
```shell
# Pod 조회
kubectl get pod
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl get pod
NAME                               READY   STATUS    RESTARTS   AGE
wordpress-746bd6d54b-mdrbp         1/1     Running   0          29m
wordpress-mysql-78488dd7d5-qxwsm   1/1     Running   0          29m

# 줄임말(Shortname)과 복수형 사용가능
kubectl get pods
kubectl get po

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
wordpress-746bd6d54b-mdrbp         1/1     Running   0          31m
wordpress-mysql-78488dd7d5-qxwsm   1/1     Running   0          31m
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl get po
NAME                               READY   STATUS    RESTARTS   AGE
wordpress-746bd6d54b-mdrbp         1/1     Running   0          31m
wordpress-mysql-78488dd7d5-qxwsm   1/1     Running   0          31m

# 여러 TYPE 입력
kubectl get pod,service
kubectl get po,svc

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl get pod,service
NAME                                   READY   STATUS    RESTARTS   AGE
pod/wordpress-746bd6d54b-mdrbp         1/1     Running   0          32m
pod/wordpress-mysql-78488dd7d5-qxwsm   1/1     Running   0          32m

NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP        20h
service/wordpress         NodePort    10.111.192.24   <none>        80:31103/TCP   32m
service/wordpress-mysql   ClusterIP   10.99.246.116   <none>        3306/TCP       32m

# Pod, ReplicaSet, Deployment, Service, Job 조회 => all
kubectl get all

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl get all
NAME                                   READY   STATUS    RESTARTS   AGE
pod/wordpress-746bd6d54b-mdrbp         1/1     Running   0          32m
pod/wordpress-mysql-78488dd7d5-qxwsm   1/1     Running   0          32m

NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP        20h
service/wordpress         NodePort    10.111.192.24   <none>        80:31103/TCP   32m
service/wordpress-mysql   ClusterIP   10.99.246.116   <none>        3306/TCP       32m

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/wordpress         1/1     1            1           32m
deployment.apps/wordpress-mysql   1/1     1            1           32m

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/wordpress-746bd6d54b         1         1         1       32m
replicaset.apps/wordpress-mysql-78488dd7d5   1         1         1       32m


```
kubectl get pod -o wide
```shell
# 결과 포멧 변경
kubectl get pod -o wide
kubectl get pod -o yaml
kubectl get pod -o json

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl get pod -o wide
NAME                               READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
wordpress-746bd6d54b-mdrbp         1/1     Running   0          33m   10.244.0.13   minikube   <none>           <none>
wordpress-mysql-78488dd7d5-qxwsm   1/1     Running   0          33m   10.244.0.14   minikube   <none>           <none>

```
kubectl get pod -o yaml

```shell
oadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl get pod -o yaml
```
```yaml

apiVersion: v1
items:
- apiVersion: v1
  kind: Pod
  metadata:
    creationTimestamp: "2023-11-28T03:56:33Z"
    generateName: wordpress-746bd6d54b-
    labels:
      app: wordpress
      pod-template-hash: 746bd6d54b
      tier: frontend
    name: wordpress-746bd6d54b-mdrbp
    namespace: default
    ownerReferences:
    - apiVersion: apps/v1
      blockOwnerDeletion: true
      controller: true
      kind: ReplicaSet
      name: wordpress-746bd6d54b
      uid: 8ab7558a-0cfe-420b-9365-3cd42e712695
    resourceVersion: "59131"
    uid: 8f36749a-2a47-485e-b914-d506c1c5418e
  spec:
    containers:
    - env:
      - name: WORDPRESS_DB_HOST
        value: wordpress-mysql
      - name: WORDPRESS_DB_NAME
        value: wordpress
      - name: WORDPRESS_DB_USER
        value: root
      - name: WORDPRESS_DB_PASSWORD
        value: password
      image: wordpress:5.9.1-php8.1-apache
      imagePullPolicy: IfNotPresent
      name: wordpress
      ports:
      - containerPort: 80
        name: wordpress
        protocol: TCP
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      volumeMounts:
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: kube-api-access-4gcp9
        readOnly: true
    dnsPolicy: ClusterFirst
    enableServiceLinks: true
    nodeName: minikube
    preemptionPolicy: PreemptLowerPriority
    priority: 0
    restartPolicy: Always
    schedulerName: default-scheduler
    securityContext: {}
    serviceAccount: default
    serviceAccountName: default
    terminationGracePeriodSeconds: 30
    tolerations:
    - effect: NoExecute
      key: node.kubernetes.io/not-ready
      operator: Exists
      tolerationSeconds: 300
    - effect: NoExecute
      key: node.kubernetes.io/unreachable
      operator: Exists
      tolerationSeconds: 300
    volumes:
    - name: kube-api-access-4gcp9
      projected:
        defaultMode: 420
        sources:
        - serviceAccountToken:
            expirationSeconds: 3607
            path: token
        - configMap:
            items:
            - key: ca.crt
              path: ca.crt
            name: kube-root-ca.crt
        - downwardAPI:
            items:
            - fieldRef:
                apiVersion: v1
                fieldPath: metadata.namespace
              path: namespace
  status:
    conditions:
    - lastProbeTime: null
      lastTransitionTime: "2023-11-28T03:56:33Z"
      status: "True"
      type: Initialized
    - lastProbeTime: null
      lastTransitionTime: "2023-11-28T03:56:35Z"
      status: "True"
      type: Ready
    - lastProbeTime: null
      lastTransitionTime: "2023-11-28T03:56:35Z"
      status: "True"
      type: ContainersReady
    - lastProbeTime: null
      lastTransitionTime: "2023-11-28T03:56:33Z"
      status: "True"
      type: PodScheduled
    containerStatuses:
    - containerID: docker://0ad19fb4d2839bc1ccbe2e5717268f34059e9cdc96e933c1b6a41b6f1e99c7a5
      image: wordpress:5.9.1-php8.1-apache
      imageID: docker-pullable://wordpress@sha256:54c1c0d69afb68aa972d2be131265200b799dbfabd9eda50eb0b3ff17023b1c4
      lastState: {}
      name: wordpress
      ready: true
      restartCount: 0
      started: true
      state:
        running:
          startedAt: "2023-11-28T03:56:34Z"
    hostIP: 192.168.49.2
    phase: Running
    podIP: 10.244.0.13
    podIPs:
    - ip: 10.244.0.13
    qosClass: BestEffort
    startTime: "2023-11-28T03:56:33Z"
- apiVersion: v1
  kind: Pod
  metadata:
    creationTimestamp: "2023-11-28T03:56:33Z"
    generateName: wordpress-mysql-78488dd7d5-
    labels:
      app: wordpress
      pod-template-hash: 78488dd7d5
      tier: mysql
    name: wordpress-mysql-78488dd7d5-qxwsm
    namespace: default
    ownerReferences:
    - apiVersion: apps/v1
      blockOwnerDeletion: true
      controller: true
      kind: ReplicaSet
      name: wordpress-mysql-78488dd7d5
      uid: b31b6eba-1e57-4ce7-a821-f0a9b2817d80
    resourceVersion: "59127"
    uid: a87ecfc9-5b1d-4a1e-868b-a543a0abf1d4
  spec:
    containers:
    - env:
      - name: MYSQL_DATABASE
        value: wordpress
      - name: MYSQL_ROOT_PASSWORD
        value: password
      image: mariadb:10.7
      imagePullPolicy: IfNotPresent
      name: mysql
      ports:
      - containerPort: 3306
        name: mysql
        protocol: TCP
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      volumeMounts:
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: kube-api-access-9qf5l
        readOnly: true
    dnsPolicy: ClusterFirst
    enableServiceLinks: true
    nodeName: minikube
    preemptionPolicy: PreemptLowerPriority
    priority: 0
    restartPolicy: Always
    schedulerName: default-scheduler
    securityContext: {}
    serviceAccount: default
    serviceAccountName: default
    terminationGracePeriodSeconds: 30
    tolerations:
    - effect: NoExecute
      key: node.kubernetes.io/not-ready
      operator: Exists
      tolerationSeconds: 300
    - effect: NoExecute
      key: node.kubernetes.io/unreachable
      operator: Exists
      tolerationSeconds: 300
    volumes:
    - name: kube-api-access-9qf5l
      projected:
        defaultMode: 420
        sources:
        - serviceAccountToken:
            expirationSeconds: 3607
            path: token
        - configMap:
            items:
            - key: ca.crt
              path: ca.crt
            name: kube-root-ca.crt
        - downwardAPI:
            items:
            - fieldRef:
                apiVersion: v1
                fieldPath: metadata.namespace
              path: namespace
  status:
    conditions:
    - lastProbeTime: null
      lastTransitionTime: "2023-11-28T03:56:33Z"
      status: "True"
      type: Initialized
    - lastProbeTime: null
      lastTransitionTime: "2023-11-28T03:56:34Z"
      status: "True"
      type: Ready
    - lastProbeTime: null
      lastTransitionTime: "2023-11-28T03:56:34Z"
      status: "True"
      type: ContainersReady
    - lastProbeTime: null
      lastTransitionTime: "2023-11-28T03:56:33Z"
      status: "True"
      type: PodScheduled
    containerStatuses:
    - containerID: docker://f942e1b466bc60134eaf4784b1d58dc58fb1530764b20471b0342279bcba1cc6
      image: mariadb:10.7
      imageID: docker-pullable://mariadb@sha256:9a48ac9f196f3d4fd6fea2cab59a49df9e7ca459bf14b2f7b85a0e38a5454571
      lastState: {}
      name: mysql
      ready: true
      restartCount: 0
      started: true
      state:
        running:
          startedAt: "2023-11-28T03:56:34Z"
    hostIP: 192.168.49.2
    phase: Running
    podIP: 10.244.0.14
    podIPs:
    - ip: 10.244.0.14
    qosClass: BestEffort
    startTime: "2023-11-28T03:56:33Z"
kind: List
metadata:
  resourceVersion: ""
```
kubectl get pod -o json
```shell
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl get pod -o json
```
```json
{
    "apiVersion": "v1",
    "items": [
        {
            "apiVersion": "v1",
            "kind": "Pod",
            "metadata": {
                "creationTimestamp": "2023-11-28T03:56:33Z",
                "generateName": "wordpress-746bd6d54b-",
                "labels": {
                    "app": "wordpress",
                    "pod-template-hash": "746bd6d54b",
                    "tier": "frontend"
                },
                "name": "wordpress-746bd6d54b-mdrbp",
                "namespace": "default",
                "ownerReferences": [
                    {
                        "apiVersion": "apps/v1",
                        "blockOwnerDeletion": true,
                        "controller": true,
                        "kind": "ReplicaSet",
                        "name": "wordpress-746bd6d54b",
                        "uid": "8ab7558a-0cfe-420b-9365-3cd42e712695"
                    }
                ],
                "resourceVersion": "59131",
                "uid": "8f36749a-2a47-485e-b914-d506c1c5418e"
            },
            "spec": {
                "containers": [
                    {
                        "env": [
                            {
                                "name": "WORDPRESS_DB_HOST",
                                "value": "wordpress-mysql"
                            },
                            {
                                "name": "WORDPRESS_DB_NAME",
                                "value": "wordpress"
                            },
                            {
                                "name": "WORDPRESS_DB_USER",
                                "value": "root"
                            },
                            {
                                "name": "WORDPRESS_DB_PASSWORD",
                                "value": "password"
                            }
                        ],
                        "image": "wordpress:5.9.1-php8.1-apache",
                        "imagePullPolicy": "IfNotPresent",
                        "name": "wordpress",
                        "ports": [
                            {
                                "containerPort": 80,
                                "name": "wordpress",
                                "protocol": "TCP"
                            }
                        ],
                        "resources": {},
                        "terminationMessagePath": "/dev/termination-log",
                        "terminationMessagePolicy": "File",
                        "volumeMounts": [
                            {
                                "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
                                "name": "kube-api-access-4gcp9",
                                "readOnly": true
                            }
                        ]
                    }
                ],
                "dnsPolicy": "ClusterFirst",
                "enableServiceLinks": true,
                "nodeName": "minikube",
                "preemptionPolicy": "PreemptLowerPriority",
                "priority": 0,
                "restartPolicy": "Always",
                "schedulerName": "default-scheduler",
                "securityContext": {},
                "serviceAccount": "default",
                "serviceAccountName": "default",
                "terminationGracePeriodSeconds": 30,
                "tolerations": [
                    {
                        "effect": "NoExecute",
                        "key": "node.kubernetes.io/not-ready",
                        "operator": "Exists",
                        "tolerationSeconds": 300
                    },
                    {
                        "effect": "NoExecute",
                        "key": "node.kubernetes.io/unreachable",
                        "operator": "Exists",
                        "tolerationSeconds": 300
                    }
                ],
                "volumes": [
                    {
                        "name": "kube-api-access-4gcp9",
                        "projected": {
                            "defaultMode": 420,
                            "sources": [
                                {
                                    "serviceAccountToken": {
                                        "expirationSeconds": 3607,
                                        "path": "token"
                                    }
                                },
                                {
                                    "configMap": {
                                        "items": [
                                            {
                                                "key": "ca.crt",
                                                "path": "ca.crt"
                                            }
                                        ],
                                        "name": "kube-root-ca.crt"
                                    }
                                },
                                {
                                    "downwardAPI": {
                                        "items": [
                                            {
                                                "fieldRef": {
                                                    "apiVersion": "v1",
                                                    "fieldPath": "metadata.namespace"
                                                },
                                                "path": "namespace"
                                            }
                                        ]
                                    }
                                }
                            ]
                        }
                    }
                ]
            },
            "status": {
                "conditions": [
                    {
                        "lastProbeTime": null,
                        "lastTransitionTime": "2023-11-28T03:56:33Z",
                        "status": "True",
                        "type": "Initialized"
                    },
                    {
                        "lastProbeTime": null,
                        "lastTransitionTime": "2023-11-28T03:56:35Z",
                        "status": "True",
                        "type": "Ready"
                    },
                    {
                        "lastProbeTime": null,
                        "lastTransitionTime": "2023-11-28T03:56:35Z",
                        "status": "True",
                        "type": "ContainersReady"
                    },
                    {
                        "lastProbeTime": null,
                        "lastTransitionTime": "2023-11-28T03:56:33Z",
                        "status": "True",
                        "type": "PodScheduled"
                    }
                ],
                "containerStatuses": [
                    {
                        "containerID": "docker://0ad19fb4d2839bc1ccbe2e5717268f34059e9cdc96e933c1b6a41b6f1e99c7a5",
                        "image": "wordpress:5.9.1-php8.1-apache",
                        "imageID": "docker-pullable://wordpress@sha256:54c1c0d69afb68aa972d2be131265200b799dbfabd9eda50eb0b3ff17023b1c4",
                        "lastState": {},
                        "name": "wordpress",
                        "ready": true,
                        "restartCount": 0,
                        "started": true,
                        "state": {
                            "running": {
                                "startedAt": "2023-11-28T03:56:34Z"
                            }
                        }
                    }
                ],
                "hostIP": "192.168.49.2",
                "phase": "Running",
                "podIP": "10.244.0.13",
                "podIPs": [
                    {
                        "ip": "10.244.0.13"
                    }
                ],
                "qosClass": "BestEffort",
                "startTime": "2023-11-28T03:56:33Z"
            }
        },
        {
            "apiVersion": "v1",
            "kind": "Pod",
            "metadata": {
                "creationTimestamp": "2023-11-28T03:56:33Z",
                "generateName": "wordpress-mysql-78488dd7d5-",
                "labels": {
                    "app": "wordpress",
                    "pod-template-hash": "78488dd7d5",
                    "tier": "mysql"
                },
                "name": "wordpress-mysql-78488dd7d5-qxwsm",
                "namespace": "default",
                "ownerReferences": [
                    {
                        "apiVersion": "apps/v1",
                        "blockOwnerDeletion": true,
                        "controller": true,
                        "kind": "ReplicaSet",
                        "name": "wordpress-mysql-78488dd7d5",
                        "uid": "b31b6eba-1e57-4ce7-a821-f0a9b2817d80"
                    }
                ],
                "resourceVersion": "59127",
                "uid": "a87ecfc9-5b1d-4a1e-868b-a543a0abf1d4"
            },
            "spec": {
                "containers": [
                    {
                        "env": [
                            {
                                "name": "MYSQL_DATABASE",
                                "value": "wordpress"
                            },
                            {
                                "name": "MYSQL_ROOT_PASSWORD",
                                "value": "password"
                            }
                        ],
                        "image": "mariadb:10.7",
                        "imagePullPolicy": "IfNotPresent",
                        "name": "mysql",
                        "ports": [
                            {
                                "containerPort": 3306,
                                "name": "mysql",
                                "protocol": "TCP"
                            }
                        ],
                        "resources": {},
                        "terminationMessagePath": "/dev/termination-log",
                        "terminationMessagePolicy": "File",
                        "volumeMounts": [
                            {
                                "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
                                "name": "kube-api-access-9qf5l",
                                "readOnly": true
                            }
                        ]
                    }
                ],
                "dnsPolicy": "ClusterFirst",
                "enableServiceLinks": true,
                "nodeName": "minikube",
                "preemptionPolicy": "PreemptLowerPriority",
                "priority": 0,
                "restartPolicy": "Always",
                "schedulerName": "default-scheduler",
                "securityContext": {},
                "serviceAccount": "default",
                "serviceAccountName": "default",
                "terminationGracePeriodSeconds": 30,
                "tolerations": [
                    {
                        "effect": "NoExecute",
                        "key": "node.kubernetes.io/not-ready",
                        "operator": "Exists",
                        "tolerationSeconds": 300
                    },
                    {
                        "effect": "NoExecute",
                        "key": "node.kubernetes.io/unreachable",
                        "operator": "Exists",
                        "tolerationSeconds": 300
                    }
                ],
                "volumes": [
                    {
                        "name": "kube-api-access-9qf5l",
                        "projected": {
                            "defaultMode": 420,
                            "sources": [
                                {
                                    "serviceAccountToken": {
                                        "expirationSeconds": 3607,
                                        "path": "token"
                                    }
                                },
                                {
                                    "configMap": {
                                        "items": [
                                            {
                                                "key": "ca.crt",
                                                "path": "ca.crt"
                                            }
                                        ],
                                        "name": "kube-root-ca.crt"
                                    }
                                },
                                {
                                    "downwardAPI": {
                                        "items": [
                                            {
                                                "fieldRef": {
                                                    "apiVersion": "v1",
                                                    "fieldPath": "metadata.namespace"
                                                },
                                                "path": "namespace"
                                            }
                                        ]
                                    }
                                }
                            ]
                        }
                    }
                ]
            },
            "status": {
                "conditions": [
                    {
                        "lastProbeTime": null,
                        "lastTransitionTime": "2023-11-28T03:56:33Z",
                        "status": "True",
                        "type": "Initialized"
                    },
                    {
                        "lastProbeTime": null,
                        "lastTransitionTime": "2023-11-28T03:56:34Z",
                        "status": "True",
                        "type": "Ready"
                    },
                    {
                        "lastProbeTime": null,
                        "lastTransitionTime": "2023-11-28T03:56:34Z",
                        "status": "True",
                        "type": "ContainersReady"
                    },
                    {
                        "lastProbeTime": null,
                        "lastTransitionTime": "2023-11-28T03:56:33Z",
                        "status": "True",
                        "type": "PodScheduled"
                    }
                ],
                "containerStatuses": [
                    {
                        "containerID": "docker://f942e1b466bc60134eaf4784b1d58dc58fb1530764b20471b0342279bcba1cc6",
                        "image": "mariadb:10.7",
                        "imageID": "docker-pullable://mariadb@sha256:9a48ac9f196f3d4fd6fea2cab59a49df9e7ca459bf14b2f7b85a0e38a5454571",
                        "lastState": {},
                        "name": "mysql",
                        "ready": true,
                        "restartCount": 0,
                        "started": true,
                        "state": {
                            "running": {
                                "startedAt": "2023-11-28T03:56:34Z"
                            }
                        }
                    }
                ],
                "hostIP": "192.168.49.2",
                "phase": "Running",
                "podIP": "10.244.0.14",
                "podIPs": [
                    {
                        "ip": "10.244.0.14"
                    }
                ],
                "qosClass": "BestEffort",
                "startTime": "2023-11-28T03:56:33Z"
            }
        }
    ],
    "kind": "List",
    "metadata": {
        "resourceVersion": ""
    }
}
```

```shell
# Label 조회
kubectl get pod --show-labels

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl get pod --show-labels
NAME                               READY   STATUS    RESTARTS   AGE   LABELS
wordpress-746bd6d54b-mdrbp         1/1     Running   0          43m   app=wordpress,pod-template-hash=746bd6d54b,tier=frontend
wordpress-mysql-78488dd7d5-qxwsm   1/1     Running   0          43m   app=wordpress,pod-template-hash=78488dd7d5,tier=mysql
```

#### 리스스 상세보기(describe)
쿠버네티스에 선언된 리소스의 상세한 상태를 확인하는 명령어이다.
특정 리소스의 상태가 궁금하거나 생성이 실패한 이유를 확인할 때 주로 사용한다.
```shell
# Pod 조회로 이름 검색
kubectl get pod

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl get pod
NAME                               READY   STATUS    RESTARTS   AGE
wordpress-746bd6d54b-mdrbp         1/1     Running   0          47m
wordpress-mysql-78488dd7d5-qxwsm   1/1     Running   0          47m

# 조회한 이름으로 상세 확인
kubectl describe pod/wordpress-5f59577d4d-8t2dg # 환경마다 이름이 다릅니다

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl describe pod/wordpress-746bd6d54b-mdrbp
Name:             wordpress-746bd6d54b-mdrbp
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Tue, 28 Nov 2023 12:56:33 +0900
Labels:           app=wordpress
                  pod-template-hash=746bd6d54b
                  tier=frontend
Annotations:      <none>
Status:           Running
IP:               10.244.0.13
IPs:
  IP:           10.244.0.13
Controlled By:  ReplicaSet/wordpress-746bd6d54b
Containers:
  wordpress:
    Container ID:   docker://0ad19fb4d2839bc1ccbe2e5717268f34059e9cdc96e933c1b6a41b6f1e99c7a5
    Image:          wordpress:5.9.1-php8.1-apache
    Image ID:       docker-pullable://wordpress@sha256:54c1c0d69afb68aa972d2be131265200b799dbfabd9eda50eb0b3ff17023b1c4
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 28 Nov 2023 12:56:34 +0900
    Ready:          True
    Restart Count:  0
    Environment:
      WORDPRESS_DB_HOST:      wordpress-mysql
      WORDPRESS_DB_NAME:      wordpress
      WORDPRESS_DB_USER:      root
      WORDPRESS_DB_PASSWORD:  password
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-4gcp9 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-4gcp9:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  48m   default-scheduler  Successfully assigned default/wordpress-746bd6d54b-mdrbp to minikube
  Normal  Pulled     48m   kubelet            Container image "wordpress:5.9.1-php8.1-apache" already present on machine
  Normal  Created    48m   kubelet            Created container wordpress
  Normal  Started    48m   kubelet            Started container wordpress
```


#### 리소스 제거(delete)
쿠버네티스에 선언된 리소스를 제거하는 명령어
```shell
# Pod 조회로 이름 검색
kubectl get pod


roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl get pod
NAME                               READY   STATUS    RESTARTS   AGE
wordpress-746bd6d54b-mdrbp         1/1     Running   0          51m
wordpress-mysql-78488dd7d5-qxwsm   1/1     Running   0          51m

# 조회한 Pod 제거
kubectl delete pod/wordpress-5f59577d4d-8t2dg

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl delete pod/wordpress-746bd6d54b-mdrbp
pod "wordpress-746bd6d54b-mdrbp" deleted
```

#### 컨테이너 로그 조회(logs)
컨테이너의 로그를 확인하는 명령어
```shell

# Pod 조회로 이름 검색
kubectl get pod

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl get pod
NAME                               READY   STATUS    RESTARTS   AGE
wordpress-746bd6d54b-mcmmk         1/1     Running   0          75s
wordpress-mysql-78488dd7d5-qxwsm   1/1     Running   0          54m

# 조회한 Pod 로그조회
kubectl logs wordpress-5f59577d4d-8t2dg

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl logs wordpress-746bd6d54b-mcmmk
WordPress not found in /var/www/html - copying now...
Complete! WordPress has been successfully copied to /var/www/html
No 'wp-config.php' found in /var/www/html, but 'WORDPRESS_...' variables supplied; copying 'wp-config-docker.php' (WORDPRESS_DB_HOST WORDPRESS_DB_NAME WORDPRESS_DB_PASSWORD WORDPRESS_DB_USER WORDPRESS_MYSQL_PORT WORDPRESS_MYSQL_PORT_3306_TCP WORDPRESS_MYSQL_PORT_3306_TCP_ADDR WORDPRESS_MYSQL_PORT_3306_TCP_PORT WORDPRESS_MYSQL_PORT_3306_TCP_PROTO WORDPRESS_MYSQL_SERVICE_HOST WORDPRESS_MYSQL_SERVICE_PORT WORDPRESS_PORT WORDPRESS_PORT_80_TCP WORDPRESS_PORT_80_TCP_ADDR WORDPRESS_PORT_80_TCP_PORT WORDPRESS_PORT_80_TCP_PROTO WORDPRESS_SERVICE_HOST WORDPRESS_SERVICE_PORT)
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 10.244.0.15. Set the 'ServerName' directive globally to suppress this message
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 10.244.0.15. Set the 'ServerName' directive globally to suppress this message
[Tue Nov 28 04:49:21.086435 2023] [mpm_prefork:notice] [pid 1] AH00163: Apache/2.4.52 (Debian) PHP/8.1.3 configured -- resuming normal operations
[Tue Nov 28 04:49:21.086483 2023] [core:notice] [pid 1] AH00094: Command line: 'apache2 -D FOREGROUND'

# 실시간 로그 보기
kubectl logs -f wordpress-5f59577d4d-8t2dg

```


#### 컨테이너 명령어 전달(exec)
컨테이너에 접속하는 명령어

```shell
# Pod 조회로 이름 검색
kubectl get pod

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl get pod
NAME                               READY   STATUS    RESTARTS   AGE
wordpress-746bd6d54b-mcmmk         1/1     Running   0          2m52s
wordpress-mysql-78488dd7d5-qxwsm   1/1     Running   0          55m

# 조회한 Pod의 컨테이너에 접속
kubectl exec -it wordpress-5f59577d4d-8t2dg -- bash

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl exec -it wordpress-746bd6d54b-mcmmk -- bash
root@wordpress-746bd6d54b-mcmmk:/var/www/html#
```

#### 설정 관리 (config)
kubectl 은 여러 개의 쿠버네티스 클러스터를 컨텍스트로 설정하고 필요에 따라 선택할 수 있다. 
```shell
# 현재 컨텍스트 확인
kubectl config current-context

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl config current-context
minikube

# 컨텍스트 설정
kubectl config use-context minikube

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl config use-context minikube
Switched to context "minikube".

```
#### 리소스 제거

```shell
kubectl delete -f https://subicura.com/k8s/code/guide/index/wordpress-k8s.yml

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl delete -f https://subicura.com/k8s/code/guide/index/wordpress-k8s.yml
deployment.apps "wordpress-mysql" deleted
service "wordpress-mysql" deleted
deployment.apps "wordpress" deleted
service "wordpress" deletedku
```