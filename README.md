# Assignment 4
## Exercise 1
Provide the YAML that allows you to manage a custom resource of type "Fruit" with Kubernetes: We call this yaml fruit.yaml 

apiVersion: apiextensions.k8s.io/v1

kind: CustomResourceDefinition

metadata:

`  `name: fruits.hy548.csd.uoc.gr

spec:

`  `group: hy548.csd.uoc.gr

`  `versions:

`    `- name: v1

`      `served: true

`      `storage: true

`      `schema:

`        `openAPIV3Schema:

`          `type: object

`          `properties:

`            `spec:

`              `type: object

`              `properties:

`                `origin:

`                  `type: string

`                `count:

`                  `type: integer

`                `grams:

`                  `type: integer

`  `scope: Namespaced

`  `names:

`    `plural: fruits

`    `singular: fruit

`    `kind: Fruit

`    `shortNames:

`      `- frt


And the fruit-inst.yaml is:

apiVersion: hy548.csd.uoc.gr/v1

kind: Fruit

metadata:

`  `name: apple

spec:

`  `origin: Krousonas

`  `count: 3

`  `grams: 980

Then we provide the kubectl commands needed to:

1. Install the custom resource.

   minikube start

   kubectl apply -f fruit.yaml

1. Create the above instance.

kubectl apply -f fruit-inst.yaml

1. Return the new instance in YAML format.

kubectl get fruit apple -o yaml

1. Return a list of all available instances.

kubectl get fruits since fruits is defined as the plural of fruit in fruit.yaml


## Exercise 2
Extend the example so that:

1. The executable controller.py runs inside a container. Provide the Dockerfile. Build and upload the new container to Docker Hub.

   The Dockerfile is this:

   FROM python:3.9-slim

   WORKDIR /app

   COPY requirements.txt requirements.txt

   RUN pip install --no-cache-dir -r requirements.txt

   COPY . .

   CMD ["python", "controller.py"]


   To build and upload the new container to Docker Hub we use these commands:

   docker build -t jimaandron/greeting-controller:latest .

   docker push jimaandron/greeting-controller:latest

   The repo is this <https://hub.docker.com/r/jimaandron/greeting-controller>

1. Provide greeting-controller.yaml, which will create a Deployment with the container you made. Make sure the necessary permissions are set so the controller can read the "Greeting" CRDs and create the corresponding Deployments with the hello-kubernetes container in all namespaces.

   The greeting-controller.yaml is this:

   apiVersion: apps/v1

   kind: Deployment

   metadata:

   `  `name: greeting-controller

   spec:

   `  `replicas: 1

   `  `selector:

   `    `matchLabels:

   `      `app: greeting-controller

   `  `template:

   `    `metadata:

   `      `labels:

   `        `app: greeting-controller

   `    `spec:

   `      `containers:

   `      `- name: greeting-controller

   `        `image: jimaandron/greeting-controller:latest

   `        `imagePullPolicy: Always

   `        `env:

   `        `- name: KUBERNETES\_SERVICE\_HOST

   `          `value: "kubernetes.default.svc"

   `        `- name: KUBERNETES\_SERVICE\_PORT

   `          `value: "443"

   `      `serviceAccountName: greeting-controller-sa

   ---

   apiVersion: v1

   kind: ServiceAccount

   metadata:

   `  `name: greeting-controller-sa

   `  `namespace: default

   ---

   apiVersion: rbac.authorization.k8s.io/v1

   kind: ClusterRole

   metadata:

   `  `name: greeting-controller-clusterrole

   rules:

   - apiGroups: [""]

   `  `resources: ["pods", "services", "namespaces"]

   `  `verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

   - apiGroups: ["apps"]

   `  `resources: ["deployments"]

   `  `verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

   - apiGroups: ["hy548.csd.uoc.gr"]

   `  `resources: ["greetings"]

   `  `verbs: ["get", "list", "watch"]

   ---

   apiVersion: rbac.authorization.k8s.io/v1

   kind: ClusterRoleBinding

   metadata:

   `  `name: greeting-controller-clusterrolebinding

   subjects:

   - kind: ServiceAccount

   `  `name: greeting-controller-sa

   `  `namespace: default

   roleRef:

   `  `kind: ClusterRole

   `  `name: greeting-controller-clusterrole

   `  `apiGroup: rbac.authorization.k8s.io



   Provide the commands you used to verify that the deployment works correctly.


   kubectl apply -f greeting-crd.yaml

   kubectl apply -f greeting-cntrl.yaml

   kubectl get deployments

   kubectl logs deployment/greeting-controller

   kubectl get greetings

   In logs and in get deployments we see that greetings controller is running. But in get greetings we see that there is nothing there. So after we apply the hello-world.yaml too, we see :

   NAME           AGE

   hello-to-all   52s

   hello-world    52s

   Which is correct!
   ## Exercise 3
   The example code for the webhooks is available on GitHub (https://github.com/chazapis/hy548). Extend the example so that:

   1. The executable controller.py runs inside a container. Provide the Dockerfile. Build and upload the new container to Docker Hub.

      The Dockerfile is this:

      FROM python:3.9-slim

      WORKDIR /app

      COPY requirements.txt requirements.txt

      RUN pip install --no-cache-dir -r requirements.txt

      COPY . .

      # Expose the port the webhook listens on

      EXPOSE 8000

      CMD ["python", "controller.py"]

      To build and upload the new container to Docker Hub, we use these:

      docker build -t jimaandron/webhook-controller:latest .

      docker push jimaandron/webhook-controller:latest


   1. The webhook.yaml should use the new container instead of the proxy with Nginx. Provide the new webhook.yaml.

      The new webhook.yaml is this:

      apiVersion: v1

      kind: Namespace

      metadata:

      `  `name: custom-label-injector

      `  `labels:

      `    `app: custom-label-injector

      ---

      apiVersion: cert-manager.io/v1

      kind: Issuer

      metadata:

      `  `name: issuer-selfsigned

      `  `namespace: custom-label-injector

      `  `labels:

      `    `app: custom-label-injector

      spec:

      `  `selfSigned: {}

      ---

      apiVersion: cert-manager.io/v1

      kind: Certificate

      metadata:

      `  `name: controller-certificate

      `  `namespace: custom-label-injector

      `  `labels:

      `    `app: custom-label-injector

      spec:

      `  `secretName: controller-certificate

      `  `duration: 87600h

      `  `commonName: controller.custom-label-injector.svc

      `  `dnsNames:

      `  `- controller.custom-label-injector.svc

      `  `privateKey:

      `    `algorithm: RSA

      `    `size: 2048

      `  `issuerRef:

      `    `name: issuer-selfsigned

      ---

      apiVersion: v1

      kind: Service

      metadata:

      `  `name: controller

      `  `namespace: custom-label-injector

      `  `labels:

      `    `app: custom-label-injector

      spec:

      `  `type: ClusterIP

      `  `ports:

      `    `- port: 443

      `      `name: https

      `  `selector:

      `    `app: custom-label-injector

      ---

      apiVersion: v1

      kind: ConfigMap

      metadata:

      `  `name: controller-proxy-config

      `  `namespace: custom-label-injector

      `  `labels:

      `    `app: custom-label-injector

      data:

      `  `default.conf: |

      `    `log\_format custom '$remote\_addr - $sent\_http\_x\_log\_user [$time\_local] "$request" '

      `                      `'$status $body\_bytes\_sent "$http\_referer" "$http\_user\_agent"';

      `    `server {

      `        `listen 443 ssl;

      `        `server\_name controller.custom-label-injector.svc;

      `        `access\_log /var/log/nginx/access.log custom;

      `        `ssl\_certificate         /etc/ssl/keys/tls.crt;

      `        `ssl\_certificate\_key     /etc/ssl/keys/tls.key;

      `        `ssl\_session\_cache       builtin:1000 shared:SSL:10m;

      `        `ssl\_protocols           TLSv1 TLSv1.1 TLSv1.2;

      `        `ssl\_ciphers             HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;

      `        `ssl\_prefer\_server\_ciphers on;

      `        `location / {

      `            `proxy\_set\_header    Host $host;

      `            `proxy\_set\_header    X-Real-IP $remote\_addr;

      `            `proxy\_set\_header    X-Forwarded-For $proxy\_add\_x\_forwarded\_for;

      `            `proxy\_set\_header    X-Forwarded-Proto $scheme;

      `            `proxy\_pass          http://192.168.1.1:8000;

      `            `proxy\_read\_timeout  10;

      `            `proxy\_redirect      http://192.168.1.1:8000 https://controller.custom-label-injector.svc;

      `        `}

      `    `}

      ---

      apiVersion: apps/v1

      kind: Deployment

      metadata:

      `  `name: controller

      `  `namespace: custom-label-injector

      `  `labels:

      `    `app: custom-label-injector

      spec:

      `  `replicas: 1

      `  `selector:

      `    `matchLabels:

      `      `app: custom-label-injector

      `  `template:

      `    `metadata:

      `      `labels:

      `        `app: custom-label-injector

      `    `spec:

      `      `containers:

      `      `- image: jimaandron/webhook-controller:latest

      `        `name: webhook

      `        `ports:

      `        `- containerPort: 8000

      `        `env:

      `        `- name: CUSTOM\_LABEL

      `          `value: "1"

      `        `ports:

      `        `- containerPort: 443

      `          `name: https

      `        `volumeMounts:

      `        `- name: controller-certificate-volume

      `          `mountPath: /etc/ssl/keys

      `          `readOnly: true

      `        `- name: controller-proxy-config-volume

      `          `mountPath: /etc/nginx/conf.d/default.conf

      `          `subPath: default.conf

      `      `volumes:

      `      `- name: controller-certificate-volume

      `        `secret:

      `          `secretName: controller-certificate

      `      `- name: controller-proxy-config-volume

      `        `configMap:

      `          `name: controller-proxy-config

      `          `defaultMode: 0644

      ---

      apiVersion: admissionregistration.k8s.io/v1

      kind: MutatingWebhookConfiguration

      metadata:

      `  `name: custom-label-injector

      `  `namespace: custom-label-injector

      `  `labels:

      `    `app: custom-label-injector

      `  `annotations:

      `    `cert-manager.io/inject-ca-from: custom-label-injector/controller-certificate

      webhooks:

      `  `- name: controller.custom-label-injector.svc

      `    `clientConfig:

      `      `service:

      `        `name: controller

      `        `namespace: custom-label-injector

      `        `path: "/mutate"

      `    `rules:

      `      `- operations: ["CREATE"]

      `        `apiGroups: ["\*"]

      `        `apiVersions: ["\*"]

      `        `resources: ["pods", "deployments"]

      `    `namespaceSelector:

      `      `matchLabels:

      `        `custom-label-injector: enabled

      `    `admissionReviewVersions: ["v1", "v1beta1"]

      `    `sideEffects: None

      `    `failurePolicy: Fail


      Provide the commands you used to verify the webhook works correctly.

      I use Windows so firstly I downloaded helm release from this <https://github.com/helm/helm/releases>

      And then :

      >windows-amd64\helm.exe repo add jetstack https://charts.jetstack.io

      >windows-amd64\helm.exe install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version v1.7.2 --set installCRDs=true

      Then 

      kubectl create namespace test

      kubectl label namespace test custom-label-injector=enabled

      Then I created a simple test.yaml like this:

      apiVersion: v1

      kind: Pod

      metadata:

      `  `name: test-pod

      `  `namespace: test

      spec:

      `  `containers:

      `  `- name: nginx

      `    `image: nginx

      And I ran this to verify that webhook is running

      kubectl apply -f test.yaml

      kubectl get pod test-pod -n test --show-labels
