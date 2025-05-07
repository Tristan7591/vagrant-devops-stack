# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # ---------- BOX ----------
  config.vm.box = "ubuntu/jammy64"

  # ---------- HARDWARE ----------
  config.vm.provider "virtualbox" do |vb|
    vb.name   = "devops-lab"
    vb.cpus   = 4
    vb.memory = 8192
  end
    # ---------- NETWORK ----------
    config.vm.network "private_network", type: "dhcp"
    config.vm.network "forwarded_port", guest: 80, host: 8080   
    config.vm.network "forwarded_port", guest: 3000, host: 3000 
    config.vm.network "forwarded_port", guest: 8000, host: 8000 
    config.vm.network "forwarded_port", guest: 5432, host: 5432 

  # ---------- NETWORK (optionnel) ----------
  # config.vm.network "forwarded_port", guest: 8080, host: 8080

  # ---------- VERSIONS ----------
  TERRAFORM_VERSION = "1.8.5"
  KUBECTL_VERSION   = "1.28.3"
  HELM_VERSION      = "v3.14.3"
  AWSCLI_VERSION    = "2.15.32"
  EKSCTL_VERSION    = "0.168.0"
  ANSIBLE_VERSION   = "2.17.*"   # pattern apt PPA
  DOCKER_VERSION    = "5:24.*"   # docker-ce apt package

  # ---------- PROVISION ----------
  config.vm.provision "shell", inline: <<-SHELL
    set -euo pipefail

    echo "==> Mise à jour du système & paquets de base"
    sudo apt-get update -y
    sudo apt-get install -y curl unzip gnupg lsb-release ca-certificates software-properties-common git

    echo "==> Terraform #{TERRAFORM_VERSION}"
    curl -sLo terraform.zip https://releases.hashicorp.com/terraform/#{TERRAFORM_VERSION}/terraform_#{TERRAFORM_VERSION}_linux_amd64.zip
    sudo unzip -o terraform.zip -d /usr/local/bin && rm terraform.zip

    echo "==> kubectl #{KUBECTL_VERSION}"
    curl -sLo kubectl https://dl.k8s.io/release/v#{KUBECTL_VERSION}/bin/linux/amd64/kubectl
    chmod +x kubectl && sudo mv kubectl /usr/local/bin

    echo "==> Helm #{HELM_VERSION}"
    curl -sL https://get.helm.sh/helm-#{HELM_VERSION}-linux-amd64.tar.gz | tar zx
    sudo mv linux-amd64/helm /usr/local/bin && rm -rf linux-amd64

    echo "==> AWS CLI #{AWSCLI_VERSION}"
    curl -sLo awscliv2.zip https://awscli.amazonaws.com/awscli-exe-linux-x86_64-#{AWSCLI_VERSION}.zip
    unzip -q awscliv2.zip
    sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update
    rm -rf awscliv2.zip aws

    echo "==> eksctl #{EKSCTL_VERSION}"
    curl -sLo eksctl.tar.gz https://github.com/eksctl-io/eksctl/releases/download/v#{EKSCTL_VERSION}/eksctl_Linux_amd64.tar.gz
    tar -xzf eksctl.tar.gz && sudo mv eksctl /usr/local/bin && rm eksctl.tar.gz

    echo "==> Docker CE #{DOCKER_VERSION}"
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
      https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
      | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update -y
    sudo apt-get install -y docker-ce=#{DOCKER_VERSION} docker-ce-cli=#{DOCKER_VERSION} containerd.io docker-buildx-plugin docker-compose-plugin
    sudo usermod -aG docker vagrant
    sudo systemctl enable --now docker

    echo "==> Ansible #{ANSIBLE_VERSION}"
    sudo add-apt-repository --yes --update ppa:ansible/ansible
    sudo apt-get install -y ansible-core=#{ANSIBLE_VERSION}

    echo "==> Visual Studio Code (Microsoft repository)"
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://packages.microsoft.com/keys/microsoft.asc | sudo gpg --dearmor -o /etc/apt/keyrings/microsoft.gpg
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/microsoft.gpg] \
      https://packages.microsoft.com/repos/code stable main" \
      | sudo tee /etc/apt/sources.list.d/vscode.list > /dev/null
    sudo apt-get update -y
    sudo apt-get install -y code

    # ---------- VÉRIFICATION DES INSTALLATIONS ----------
    # ---------- VÉRIFICATION DES INSTALLATIONS ----------
    echo "==> Vérification finale des versions"
    echo "Terraform : $(terraform version | head -n1)"
    echo "kubectl   : $(kubectl version --client | grep -oE 'GitVersion:\"v[0-9.]+\"' | cut -d\\\" -f2)"
    echo "Helm      : $(helm version --short)"
    echo "AWS CLI   : $(aws --version)"
    echo "eksctl    : $(eksctl version)"
    echo "Docker    : $(docker --version)"
    echo "Git       : $(git --version)"
    echo "Curl      : $(curl --version | head -n1)"
    echo "Ansible   : $(ansible --version | head -n1)"
    echo "VS Code   : $(sudo -u vagrant code --version --no-sandbox --user-data-dir /tmp | head -n1)"


    echo "==> Provision terminé avec succès."
  SHELL
end