# Google Cloud Platform Lab Commands

## 1. Virtual Machine CLI Commands

### Account and Project Management
```bash
# List accounts
gcloud auth list

# List project
gcloud config list project
```

### Setting Default Values
```bash
# Set compute region and zone
gcloud config set compute/region REGION
gcloud config set compute/zone ZONE

# Get compute region and zone values
gcloud config get-value compute/zone
gcloud config get-value compute/region

# Set environment variables
export ZONE=$(gcloud config get-value compute/zone)
export PROJECT_ID=$(gcloud config get-value project)
echo -e "PROJECT ID: $PROJECT_ID\nZONE: $ZONE"
```

## 2. VM Instance Creation
```bash
gcloud compute instances create www1 \
    --zone=Zone \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "<h3>Web Server: www1</h3>" | tee /var/www/html/index.html'
```

## 3. Firewall Configuration
```bash
gcloud compute firewall-rules create www-firewall-network-lb \
    --target-tags network-lb-tag --allow tcp:80
```

## 4. Load Balancer Setup

### Create Static IP
```bash
gcloud compute addresses create network-lb-ip-1 \
    --region Region
```

### Health Check Configuration
```bash
gcloud compute http-health-checks create basic-check
```

### Target Pool Configuration
```bash
# Create target pool
gcloud compute target-pools create www-pool \
    --region Region --http-health-check basic-check

# Add instances to pool
gcloud compute target-pools add-instances www-pool \
    --instances www1,www2,www3
```

### Forwarding Rules
```bash
# Create forwarding rule
gcloud compute forwarding-rules create www-rule \
    --region Region \
    --ports 80 \
    --address network-lb-ip-1 \
    --target-pool www-pool

# View external IP
gcloud compute forwarding-rules describe www-rule --region Region
```

## 5. Load Balancer Template and Instance Groups

### Create Template
```bash
gcloud compute instance-templates create lb-backend-template \
    --region=Region \
    --network=default \
    --subnet=default \
    --tags=allow-health-check \
    --machine-type=e2-medium \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      a2ensite default-ssl
      a2enmod ssl
      vm_hostname="$(curl -H "Metadata-Flavor:Google" \
      http://169.254.169.254/computeMetadata/v1/instance/name)"
      echo "Page served from: $vm_hostname" | \
      tee /var/www/html/index.html
      systemctl restart apache2'
```

### Create Instance Group
```bash
gcloud compute instance-groups managed create lb-backend-group \
    --template=lb-backend-template --size=2 --zone=Zone
```

### Configure Firewall Rules
```bash
gcloud compute firewall-rules create fw-allow-health-check \
    --network=default \
    --action=allow \
    --direction=ingress \
    --source-ranges=130.211.0.0/22,35.191.0.0/16 \
    --target-tags=allow-health-check \
    --rules=tcp:80
```

## 6. Global Load Balancer Configuration

### Setup Global IP
```bash
gcloud compute addresses create lb-ipv4-1 \
    --ip-version=IPV4 \
    --global
```

### Configure Health Checks
```bash
gcloud compute health-checks create http http-basic-check \
    --port 80
```

### Backend Service Configuration
```bash
# Create backend service
gcloud compute backend-services create web-backend-service \
    --protocol=HTTP \
    --port-name=http \
    --health-checks=http-basic-check \
    --global

# Add instance group to backend
gcloud compute backend-services add-backend web-backend-service \
    --instance-group=lb-backend-group \
    --instance-group-zone=Zone \
    --global
```

### URL Map and Proxy Configuration
```bash
# Create URL map
gcloud compute url-maps create web-map-http \
    --default-service web-backend-service

# Create HTTP proxy
gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map-http

# Create forwarding rule
gcloud compute forwarding-rules create http-content-rule \
    --address=lb-ipv4-1\
    --global \
    --target-http-proxy=http-lb-proxy \
    --ports=80
```
