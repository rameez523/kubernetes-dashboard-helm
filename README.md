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

That's it :)
