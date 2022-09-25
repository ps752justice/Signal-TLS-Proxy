# Signal TLS Proxy

## Docker Compose

To run a Signal TLS proxy, you will need a host that has ports 80 and 443 available and a domain name that points to that host.

1. Install docker and docker-compose (`apt update && apt install docker docker-compose`)
1. Ensure your current user has access to docker (`adduser $USER docker`)
1. Clone this repository
1. `./init-certificate.sh`
1. `docker-compose up --detach`

Your proxy is now running! You can share this with the URL `https://signal.tube/#<your_host_name>`

### Updating from a previous version

If you've previously run a proxy, please update to the most recent version by pulling the most recent changes from `main`, then restarting your Docker containers:

```shell
git pull
docker-compose down
docker-compose up --detach
```

## Helm - GKE

To run a Signal TLS proxy using helm in GKE, you will need to have a cluster created and setup access to your cluster using kubectl.

Also the helm charts using Filestore in GCP. So before deploying the charts, you should have created an instances. Note the file-share name and the ip of the instance and fill it in the related values.yaml.

1. install the certbot-init shart: this will create the initial certificates and stores it in filestore persistent volumes. The files will be shared later with the nginx server.
```shell
helm install -n signal-tls-proxy certbot-init helm/certbot-init -f helm/values.yaml
export CERTBOT_JOB_SERVICE_IP=$(kubectl get svc -n signal-tls-proxy certbot-job-service -o yaml 2> /dev/null | yq .status.loadBalancer.ingress.0.ip)
echo $CERTBOT_JOB_SERVICE_IP
```
1. change the DNS record of the URL to the ip CERTBOT_JOB_SERVICE_IP.
1. wait for the job in GKE to end, meaning proper certificates are generated.
1. install the chart signal-tls-proxy
```shell
helm install -n signal-tls-proxy signal-tls-prox helm/signal-tls-proxy -f helm/values.yaml
export NGINX_SERVICE_IP=$(kubectl get svc -n signal-tls-proxy nginx-terminate-service -o yaml 2> /dev/null | yq .status.loadBalancer.ingress.0.ip)
echo $NGINX_SERVICE_IP
```
1. take the printed ip and change the DNS record for the proxy URL (proxy.ps752justice.com) to that IP.

Your proxy is now running! You can share this with the URL `https://signal.tube/#<your_host_name>`