# kubernetes-dashboard-helm
kubernetes dashboard and metrics server deployment using helm

Insallation of Kubernetes dashboard using helm chart:-
 
First need to install helm:-

=> snap install helm --classic

Then add helm repository for kubernetes dashboard and create namespace kubernetes-dashboard:-

=> helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/

=> kubectl create ns kubernetes-dashboard

Take values.yaml file from below command and save it in the file name k8sdashboard-values.yaml:-
NOTE:-
For local development, you may want to disable authentication or HTTPS. Below are the steps to disable authentication and HTTPS in the Kubernetes dashboard.

1. We need to modify the deployment of the Kubernetes dashboard to remove the argument --auto-generate-certificates and add the following extra arguments:
--enable-skip-login
--disable-settings-authorizer
--enable-insecure-login
--insecure-bind-address=0.0.0.0
2. After this change, the Kubernetes dashboard server now starts on the port 9090 for HTTP. Then we need to modify livenessProbe to use HTTP as the scheme and 9090 as the port. Port 9090 also needs to be added as containerPort.

3. we need to modify the service of the Kubernetes dashboard to open port 80 and use 9090 as the target port.

=> helm show values kubernetes-dashboard/kubernetes-dashboard > k8sdashboard-values.yaml

Install helm chart after modifying values.yaml file:-

=> helm install kubernetes-dashboard  kubernetes-dashboard/kubernetes-dashboard -f k8sdashboard-values.yaml  -n kubernetes-dashboard

Example Command:- helm install kubernetes-dashboard  kubernetes-dashboard/kubernetes-dashboard --set=service.externalPort=8080,resources.limits.cpu=200m -f k8sdashboard-values.yaml  -n kubernetes-dashboard

Example Command:- helm delete kubernetes-dashboard  kubernetes-dashboard/kubernetes-dashboard  -n kubernetes-dashboard

We didn't configure metrics to visible on kubernetes dashboard, Now we are going to upgrade helm chart by enabling metrics with this command:-

=> helm upgrade kubernetes-dashboard  kubernetes-dashboard/kubernetes-dashboard -f k8sdashboard-values.yaml  --set=metricsScraper.enabled=true -n kubernetes-dashboard

NOTE:-
Need to Configure virtual service for istio to enable public access inside kubernetes cluster.
we have file name kubernetes-dashboard.yaml for Virtual Service Istio. This will connects with Gateway Istio configuration where we set all the public URL's and TLS Secrets Configuration.

If its not working with http then edit deployment and pass these arguemnts:-

containers:
        - name: kubernetes-dashboard
          image: kubernetesui/dashboard:v2.2.0
          imagePullPolicy: Always
          ports:
            - containerPort: 8443
              protocol: TCP
            - containerPort: 9090
              protocol: TCP  
          args:
            #- --auto-generate-certificates
            - --namespace=kubernetes-dashboard
            - --enable-skip-login
            - --disable-settings-authorizer
            - --enable-insecure-login
            - --insecure-bind-address=0.0.0.0


URL  https://medium.com/@tejaswi.goudru/disable-authentication-https-in-kubernetes-dashboard-2fada478ce91



**Create an admin user for kubernetes-dashboard follow this :- 

URL https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md


Creating sample user
In this guide, we will find out how to create a new user using the Service Account mechanism of Kubernetes, grant this user admin permissions and login to Dashboard using a bearer token tied to this user.

IMPORTANT: Make sure that you know what you are doing before proceeding. Granting admin privileges to Dashboard's Service Account might be a security risk.

For each of the following snippets for ServiceAccount and ClusterRoleBinding, you should copy them to new manifest files like dashboard-adminuser.yaml and use kubectl apply -f dashboard-adminuser.yaml to create them.

Creating a Service Account
We are creating Service Account with the name admin-user in namespace kubernetes-dashboard first.

apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
Creating a ClusterRoleBinding
In most cases after provisioning the cluster using kops, kubeadm or any other popular tool, the ClusterRole cluster-admin already exists in the cluster. We can use it and create only a ClusterRoleBinding for our ServiceAccount. If it does not exist then you need to create this role first and grant required privileges manually.

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
Getting a Bearer Token
Now we need to find the token we can use to log in. Execute the following command:

kubectl -n kubernetes-dashboard create token admin-user
It should print something like:

eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXY1N253Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwMzAzMjQzYy00MDQwLTRhNTgtOGE0Ny04NDllZTliYTc5YzEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.Z2JrQlitASVwWbc-s6deLRFVk5DWD3P_vjUFXsqVSY10pbjFLG4njoZwh8p3tLxnX_VBsr7_6bwxhWSYChp9hwxznemD5x5HLtjb16kI9Z7yFWLtohzkTwuFbqmQaMoget_nYcQBUC5fDmBHRfFvNKePh_vSSb2h_aYXa8GV5AcfPQpY7r461itme1EXHQJqv-SN-zUnguDguCTjD80pFZ_CmnSE1z9QdMHPB8hoB4V68gtswR1VLa6mSYdgPwCHauuOobojALSaMc3RH7MmFUumAgguhqAkX3Omqd3rJbYOMRuMjhANqd08piDC3aIabINX6gP5-Tuuw2svnV6NYQ
Now copy the token and paste it into the Enter token field on the login screen.


That's it :)
