```
stages:          # List of stages for jobs, and their order of execution
  - install_tools
  - test
  - security
  - build
  - docker
  - deploy

install_trivy_mvn_docker_kubectl:       # Installing all the required tools. Got the commands by simply typing the command on the ubuntu instance
  stage: install_tools
  script:
    # Update package list first
    - sudo apt-get update -y
    
    # Install Java and Maven
    - sudo apt install openjdk-17-jre-headless -y
    - sudo apt install maven -y
    
    # Install Docker and fix permissions (separated commands)
    - sudo apt install docker.io -y 
    - sudo chmod 666 /var/run/docker.sock || true    # Added || true to continue even if fails
    - sudo usermod -aG docker gitlab-runner || true  # Add gitlab-runner to docker group
    
    # Install Trivy dependencies and Trivy
    - sudo apt-get install wget apt-transport-https gnupg lsb-release -y 
    - wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
    - echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
    - sudo apt-get update -y 
    - sudo apt-get install trivy -y 
  after_script:
    # Ensure Docker socket permissions are set (runs even if script fails)
    - sudo chmod 666 /var/run/docker.sock || true
  tags:
    - self-hosted

unit_testing:       # unit testing karenge
  stage: test
  script:
    - mvn test
  tags:
    - self-hosted

trivy_fs_scan:       # trivy se scan karenge
  stage: security
  script:
    - trivy fs --format table -o fs.html .
  tags:
    - self-hosted

build_app:       
  stage: build
  script:
    - mvn package
  tags:
    - self-hosted
  only:
    - main

build_and_tag_and_push_image:     
  stage: docker
  script:
    - docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD ## create variables under Setting > CI/CD > Variables
    - mvn package
    - docker build -t veronica257/boardgitlab:latest .
    - docker push veronica257/boardgitlab:latest
  tags:
    - self-hosted
  only:
    - main

k8s-deploy:
  stage: deploy
  variables:
    KUBECONFIG_PATH: /home/gitlab-runner/.kube/config
  before_script:
    # Debug: Show current user and directory
    - echo "Current user -> $(whoami)"
    
    # Remove existing .kube directory if it exists
    - rm -rf $HOME/.kube
    
    # Create fresh .kube directory with correct ownership
    - mkdir -p $HOME/.kube
    - sudo chown -R gitlab-runner:gitlab-runner $HOME/.kube
    
    # Install/Update AWS CLI
    - echo "Installing/Updating AWS CLI..."
    - sudo apt-get update
    - sudo apt-get install -y unzip
    - curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    - unzip awscliv2.zip
    - sudo ./aws/install --update
    - echo "AWS CLI version -> $(aws --version)"
    - echo "AWS caller identity:"
    - aws sts get-caller-identity
    
    # Create and configure kubeconfig with correct permissions
    - echo "$KUBECONFIG_CONTENT" | base64 -d > "$KUBECONFIG_PATH"
    - chmod 600 "$KUBECONFIG_PATH"
    - echo "Kubeconfig permissions:"
    - ls -la "$KUBECONFIG_PATH"
    
    # Export KUBECONFIG
    - export KUBECONFIG="$KUBECONFIG_PATH"
    
    # Verify setup
    - kubectl config view
    - kubectl config current-context
  script:
    - echo "Applying kubernetes manifests..."
    - kubectl apply -f deployment-service.yaml
    - echo "Checking deployment status..."
    - kubectl get deployments
    - kubectl get pods
    - kubectl get services
  after_script:
    - echo "Final Kubernetes resources:"
    - export KUBECONFIG="$KUBECONFIG_PATH"
    - kubectl get all || true
  tags:
    - self-hosted
  only:
    - main
```
