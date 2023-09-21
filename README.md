I am working towards my CKA certification and this project uses minecraft bedrock server deployed in google cloud. Using GKE autopilot to create a Kubernetes cluster and then deploying the Splunk Otel collector to pull the logs from the pods and send them over HEC (http event collector) to Splunk Enterprise deployed in my homelab.

Follow the steps in the https://github.com/wedge22/k8s-mc-splunk-otel/blob/master/gcp-guide.md to setup Google Cloud GKE using autopilot settings. The guide also explains when to use the https://github.com/wedge22/k8s-mc-splunk-otel/blob/master/minecraft-bedrock-server.yaml and create the deployment in Kubernetes. 

I followed this excellent guide from Casey West to setup the Minecraft Bedrock server.
http://caseywest.com/
