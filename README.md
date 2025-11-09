Automated Container Deployment and Administration in the Cloud

1. Project Overview

This project demonstrates automated deployment of a containerized web application on AWS using Infrastructure-as-Code (IaC) and Continuous Integration/Continuous Delivery (CI/CD).

| Component            | Technology                  | Details                                        |
|----------------------|-----------------------------|------------------------------------------------|
| Cloud Platform       | AWS                         | Region: eu-west-1 (Ireland)                    |
| IaC Tool             | Terraform (>= v1.6.0)       | Provisions 8 AWS resources                     |
| Containerization     | Docker                      | nginx:alpine base image, Port 80               |
| CI/CD                | GitHub Actions              | Automates build, push, and deployment          |
| Application          | Web Server                  | Image: ruthdsouza/ca-project-app:latest        |

---

2. Infrastructure Details (Terraform)

Networking Configuration
- VPC CIDR: 10.0.0.0/16 (65,536 IP addresses)
- Public Subnet: 10.0.1.0/24 in availability zone eu-west-1a
- Internet Gateway: Enables public internet access
- Route Table: Routes all traffic (0.0.0.0/0) to Internet Gateway

EC2 Instance
- AMI: ami-0bc691261a82b32bc (Ubuntu Server)
- Instance Type: t3.micro (2 vCPU, 1 GB RAM)
- SSH Key: Ed25519 key pair from ~/.ssh/id_ed25519.pub
- User Data Script: Installs Docker on launch

Security Groups
- Inbound Rules:
  - SSH (Port 22) - 0.0.0.0/0
  - HTTP (Port 80) - 0.0.0.0/0
- Outbound Rules: All traffic allowed


3. Docker Containerization

Dockerfile
dockerfile
FROM nginx:alpine
COPY --chown=nginx:nginx ./app/ /usr/share/nginx/html/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]


Container Registry
- Registry: Docker Hub (ruthdsouza/ca-project-app)
- Tag: latest
- Base Image: nginx:alpine (lightweight)


4. Security Implementation

- IAM Users: Created IAM user with AdministratorAccess instead of root account
- SSH Keys: Ed25519 authentication (more secure than passwords)
- Non-root Containers: Docker runs as nginx user, limiting compromise impact
- GitHub Secrets: Encrypted credential storage
- Security Groups: Firewall rules control network access


5. CI/CD Pipeline (GitHub Actions)

GitHub Secrets Required

DOCKER_USERNAME       - Docker Hub username
DOCKER_PASSWORD       - Docker Hub password
SSH_PRIVATE_KEY       - Complete Ed25519 private key with headers
EC2_PUBLIC_IP         - EC2 instance public IP address

Automated Workflow
Trigger: Push to main branch

Build Stage:
1. Checks out code from repository
2. Builds Docker image
3. Pushes to Docker Hub (ruthdsouza/ca-project-app:latest)

Deploy Stage:
1. Connects to EC2 via SSH
2. Pulls latest image from Docker Hub
3. Stops old container (`ca-web`)
4. Starts new container on port 80
5. Verifies deployment with health check


6. Deployment Process

1. Initialize and Deploy Infrastructure

cd ~/CA/terraform
terraform init
terraform plan
terraform apply -auto-approve


2. Build and Push Docker Image

sudo docker build -t ruthdsouza/ca-project-app:latest .
sudo docker login
sudo docker push ruthdsouza/ca-project-app:latest


3. Setup GitHub Repository

git init
git branch -M main
git add .
git commit -m "Initial commit - CAProject setup"
git remote add origin git@github.com:Souza2003/CAProject.git
git push -u origin main


4. Deploy Application (Automated via GitHub Actions)

git add .
git commit -m "Update application"
git push origin main


7. Challenges and Solutions

1. SSH Authentication Issues
- Challenge: Certificate-based authentication mismatch
- Solution: Generated Ed25519 key pair and configured public key authentication

2. Disk Space Management
- Challenge: "No space left on device" error
- Solution: Implemented automated Docker cleanup in deployment script

  docker system prune -a -f --volumes
  docker builder prune -a -f


8. Testing and Validation

Infrastructure Testing
- Terraform validation: `terraform validate`, `terraform plan`
- EC2 connectivity: SSH access verification
- Network testing: Security group and port accessibility

Application Testing
- Local testing: Docker container run on localhost
- Production testing: Public IP accessibility (http://34.240.23.238)
- Health check: HTTP 200 OK response verification
- Cross-browser compatibility testing


9. Monitoring

Container Health

docker ps                    # Container status
docker logs ca-web          # Application logs
docker stats                # Resource usage


System Monitoring
- CPU utilization
- Memory usage
- Disk space
- Network traffic



10. Key Achievements

1. Automated Infrastructure: Complete AWS infrastructure provisioned using Terraform
2. Containerized Application: Docker-based deployment ensuring consistency
3. Secure Authentication: SSH key-based authentication implemented
4. CI/CD Pipeline: GitHub Actions workflow automating deployment
5. Production Application: Fully functional at http://34.240.23.238
6. Problem-Solving: Resolved authentication and disk space challenges

