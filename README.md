Instructions to deploy **Jenkins** on AWS EKS Auto Mode
  1. Deploy EKS cluster through Auto Mode through AWS Console. Add the ` CoreDNS `, ` VPC CNI `, ` Kube Proxy `, `Amazon EBS CSI Driver` add ons. **NAT Gateway** is also needed since the worker nodes will be deployed in private subnets.
  2. Create a namespace. ` kubectl create ns jenkins `
  3. Deploy **PVC** & **Storage Class** using below command.

     ```
     kubectl -n jenkins apply -f pvc.yml -f storage-class.yml
     ```
  4. Deploy the **Jenkins** deployment & service using the below command.

     ```
     kubectl -n jenkins apply -f jenkins-dep.yml -f jenkins-svc.yml
     ```
  5. Deploy **Ingress**, **Ingress Class** & **Ingress Params** which will create an ALB listening on port 443. We will need to provide the arn of the certificate in the **Ingress** object. The certificate for the domain name needs to be created in ACM and DNS validation or Email validation needs to be done prior to creating these resources.
  6. Run the command ` kubectl -n jenkins apply -f ingress-all.yml `.
  7. Run ` kubectl -n jenkins get ingress ` to retrieve the ALB DNS.
  8. Point the domain name in Route 53 to the ALB as an A (alias) record.
  9. Access the app using ` https://your_domain_name `.
 10. To upgrade the application, first scale the deployment to 0, change the image and then apply the new deployment again.
     ```
     kubectl -n jenkins scale deployment jenkins --replicas=0
     kubectl -n jenkins apply -f jenkins-dep.yml
     ```

---------------------

**Helm**
To install this app using Helm, perform below steps
  1. Generate a certificate from ACM for your domain name. The certificate arn will be required in the next step since we are running ALB on port 443. Perform DNS validation or Email validation.
  2. Run the command

     ```
     helm install jenkins ./helm --namespace jenkins --create-namespace --set certificate_arn=arn_got_from_previous_step
     ```
  4. Get the ALB DNS using ` kubectl -n jenkins get ingress ` and point the domain name in Route 53 to the ALB as an A (alias) record.
  5. Access the app using ` https://your_domain_name `.
  6. Uninstall the app using ` helm uninstall jenkins --namespace jenkins `.

-----------------------------

**ArgoCD** installation
To install this app using ArgoCD, perform below steps
  1. Create a namespace. ` kubectl create ns argocd `.
  2. Apply **ArgoCD** manifest file
     
     ```
     kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
     ```
  3. Create a certificate for the preferred ArgoCD domain name in ACM, perform DNS validation or Email validation, put the arn of the certificate in ` argocd-ingress.yml ` & run

     ```
     kubectl -n argocd apply -f argocd-ingress.yml
     ```
  4. Run ` kubectl -n argocd get ingress ` to retrieve the ALB DNS. Point the domain name in Route 53 to the ALB as an A (alias) record.
  5. Access ArgoCD using ` https://argocd_domain_name `.
  6. To get the initial admin user password run the command

     ```
     kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo
     ```

-----------------------------

**Prometheus** & **Grafana** monitoring

To install the Prometheus stack, perform below steps
   1. Apply the monitoring stack manifest

      ```
      helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
      helm repo update
      helm install kube-prometheus prometheus-community/kube-prometheus-stack --create-namespace --namespace monitoring --set grafana.adminPassword='any_secure_password' --set grafana.service.type=ClusterIP
      ```

      ✅ This installs :- Prometheus, Grafana, Node Exporter, kube-state-metrics, Alertmanager

   2. Create a certificate for the preferred Grafana domain name in ACM, perform DNS validation or Email validation, put the arn of the certificate in ` monitoring-ingress.yml ` & run

      ```
      kubectl -n monitoring apply -f monitoring-ingress.yml
      ```
   3. Access Grafana using ` https://grafana_domain_name `.
   4. To get the initial admin user password run the command

      ```
      kubectl -n monitoring get secrets kube-prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo
      ```