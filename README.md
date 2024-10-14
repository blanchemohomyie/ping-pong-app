
Pinger and Ponger are 2 Go based microservices communicate securely via TLS. The pinger sends
regular signals to the ponger, which acknowledges the signals by sending a pong.


Key Features
TLS Encryption: Ensures secure communication between Pinger and Ponger using self-signed
certificates.
Containerized with Docker: Ensures easy deployment and scalability of both services.
Kubernetes-Orchestrated: Both services run on a lightweight k3d cluster, making it perfect for
development environments.
Port-forwarding: For quick local access and testing of services.


Project Setup and installation
The tools already installed for the project locally are as follow:
Docker: To containerize the apps.
Docker Desktop: To host k3d cluster
k3d: For managing the lightweight Kubernetes cluster.
TLS Certificates: Used to secure communication between services.


Running the applications

A two node agent k3d cluster was created with the following command:
``` k3d cluster create cluster --k3s-arg '--disable=servicelb@server:0' --k3s-arg
'--disable=traefik@server:0' --agents 2 ```

From the source code, the Dockerfile and Docker image of both pinger and ponger were created.
Multi-stage building was used in order to optimize the images, improve security and reduce the image
size. The first build stage compiles the Go binary and the final stage uses a lightweight Alpine
image to run the app. The command to build the image for:
    Pinger
``` docker build -t pinger:latest app -f app/pinger/Dockerfile ```
    Ponger
``` docker build -t ponger:latest app -f app/ponger/Dockerfile ```

Openssl was used to create self-signed certificate with SAN (Subject Alternative Name) to ensure the
validity of the TLS certificate. The certificates are located in the certs directory. The steps used
to create the certificates are as follow:
    Create the SAN configuration
``` [req]
distinguished_name = req_distinguished_name
req_extensions = req_ext
x509_extensions = v3_req
prompt = no

[req_distinguished_name]
C = US
ST = State
L = City
O = Organization
OU = OrgUnit
CN = ponger

[req_ext]
subjectAltName = @alt_names

[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = ponger
DNS.2 = ponger.default.svc.cluster.local ```

    Generate the certificate
``` openssl req -x509 -nodes -newkey rsa:2048 \
  -keyout server.key \
  -out server.crt \
  -days 365 \
  -config san.cnf ```


The manifests file was used to deploy the applications in k3d. This include deployment, which create
pods for pinger and ponger, services, which expose the pod internally within the cluster, and
Network Policies, which ensure communication between services.
The image of pinger and ponger was first imported to k3d cluster with the command:
``` k3d image import pinger:latest --cluster cluster ```
``` k3d image import ponger:latest --cluster cluster ```
The manifest files was deployed using the command:
``` kubectl create -f app/ponger/manifests ```
``` kubectl create -f app/pinger/manifests ```


Pinger and Ponger is up and running and could be checked through the command:
``` kubectl get pod ```
``` kubectl get svc ```
``` kubectl log <name of the pod> ```

This can be verified with the sample of the logs, displayed bellow:
   Pinger:
2024/10/14 17:21:19 Got pong
2024/10/14 17:21:20 Sent ping
   Ponger:
2024/10/14 17:21:19 Received GET /ping
2024/10/14 17:21:20 Received GET /ping

Port forward to test the application locally is as follow:
``` kubectl port-forward svc/ponger 8080:8080 ```

The application can then be accessed with
``` https://localhost:8080/ping ```


For best practice, the certificate and the key were not pushed into Github as this can lead to
serious security vulnerabilities.



Recommendation on running the application in production environment
While this project showcases a basic microservice architecture, transitioning to a production
environment requires additional measures for scalability, security, and reliability. Here are some
key enhancements:
- Secure Communications: Replace self-signed certificates with those issued by a trusted Certificate Authority (CA). Tools like Let's Encrypt or AWS Certificate Manager can automate this process.
- Efficient Traffic Management: Utilize an ingress controller like NGINX or Traefik to handle TLS termination and load balancing, improving performance and security.
- Robust Monitoring: Implement monitoring tools like Prometheus and Grafana to track service health and performance, identify issues promptly, and respond to security threats efficiently.
- Secure Image Distribution: Store Docker images in a private container registry to control access, maintain consistency, and integrate with your CI/CD pipeline.
- Simplified Kubernetes Management: Use Helm charts to package and deploy Kubernetes resources efficiently, streamlining deployments and minimizing downtime.
- Protect Sensitive Data: Store sensitive information in a secrets manager like AWS Secrets Manager or HashiCorp Vault to prevent exposure and enable automated rotation.