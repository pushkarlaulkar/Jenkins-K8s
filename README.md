Instructions to deploy **Jenkins** on AWS EKS Auto Mode
  1. Deploy EKS cluster through Auto Mode through AWS Console. Add the ` CoreDNS `, ` VPC CNI `, ` Kube Proxy ` add ons.
  2. Create a namespace. ` kubectl create ns jenkins `
  3. Deploy the `Jenkins` deployment & service using the `kubectl` command

     ` kubectl -n jenkins apply -f jenkins-dep.yml -f jenkins-svc.yml `
  4. Deploy `PVC` & `Storage Class` using below `kubectl` command.
  5. `kubectl -n jenkins apply -f pvc.yml -f storage-class.yml`.
  6. Deploy `Ingress`, `Ingress Class` & `Ingress Params` which will create an ALB listening on port 443. We will need to provide the arn of the certificate in the `Ingress` object. The certificate for the domain name needs to be created in ACM and DNS validation or Email validation needs to be done prior to creating these resources.
  7. Run the command ` kubectl -n jenkins -f ingress-all.yml `
  8. Run `kubectl -n jenkins get ingress` to retrieve the ALB DNS.
  9. Point the domain name in Route 53 to the ALB as an A (alias) record.
  10. Access the app using `https://your_domain_name`.

**Helm**
To install this app using Helm, perform below steps
  1. Generate a certificate from ACM for your domain name. The certificate arn will be required in the next step since we are running ALB on port 443.
  2. Run the command
     `helm install jenkins ./helm --namespace jenkins --create-namespace --set certificate_arn=arn_got_from_previous_step`.
  4. Get the ALB DNS using `kubectl -n jenkins get ingress` and point the domain name in Route 53 to the ALB as an A (alias) record.
  5. Access the app using `https://your_domain_name`.
  6. Uninstall the app using `helm unistall jenkins`.
