# Create a new GCP project
Click on create project in gcp console

# Enable GCP services in cli
gcloud services enable container.googleapis.com

# add the minecraft-bedrock-server.yaml to your project
nano minecraft-bedrock-server.yaml

# create the pods from the new yaml file
kubectl apply -f minecraft-bedrock-server.yaml

# add Helm repo for Otel
helm repo add splunk-otel-collector-chart https://signalfx.github.io/splunk-otel-collector-chart

# Sending data to Splunk Enterprise or Cloud custom values.yaml required
use the values.yaml

# run this to install the above
helm install my-splunk-otel-collector --values values.yaml splunk-otel-collector-chart/splunk-otel-collector

# exec into pod
kubectl exec --stdin --tty my-pod -- /bin/sh