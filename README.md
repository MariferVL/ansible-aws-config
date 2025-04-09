﻿# AWS Infrastructure Automation 🔧

[![Ansible](https://img.shields.io/badge/Ansible-2.15%2B-red.svg)](https://www.ansible.com/)
[![AWS](https://img.shields.io/badge/AWS-EC2%20%26%20RDS-orange.svg)](https://aws.amazon.com/)
[![Jenkins](https://img.shields.io/badge/Jenkins-CD%20Pipeline-blue.svg)](https://www.jenkins.io/)

Infrastructure-as-Code solution for provisioning and configuring AWS resources. Automates deployment of production-ready environments with security best practices.

## 🚀 Key Features
- **Turnkey Environment Setup**
  - EC2 instance provisioning with security hardening
  - RDS database configuration with proper networking
  - Automatic security group updates via [update_sg.sh](https://github.com/MariferVL/terraform-aws-infra/blob/feature/WN-1-infra-provisioning/scripts/update_sg.sh)

- **Smart Configuration Management**
  - Docker runtime installation & configuration
  - System package updates and optimizations
  - Environment-specific variable management

- **CI/CD Integration**
  - Jenkins pipeline for zero-downtime deployments
  - Automatic secret management with AWS SSM
  - Multi-stage rollback capabilities

- **Security First**
  - SSH key-based authentication only
  - IAM role-based access control (RBAC)
  - Encrypted secrets handling

## 🔄 Workflow Diagram
```mermaid
graph TD
    A[Terraform Apply] -->|EC2/RDS Outputs| B(Ansible Provisioning)
    B --> C{Validation}
    C -->|Success| D[Jenkins CD Pipeline]
    C -->|Failure| E[Auto Rollback]
    D --> F[Deployed Application]
```

## 🛠️ Quick Start

### Prerequisites
- Ansible 2.15+
- AWS CLI configured
- Jenkins server with proper credentials

### Configuration
1. **Clone Repository**
   ```bash
   git clone https://github.com/MariferVL/ansible-aws-config.git
   cd ansible-aws-config
   ```

2. **Set Up Inventory**
   ```ini
   # inventory.ini
   [webservers]
   <EC2_IP> ansible_ssh_private_key_file=~/.ssh/key.pem

   [databases]
   <RDS_ENDPOINT>
   ```

3. **Run Playbook**
   ```bash
   ansible-playbook deploy.yml \
     -e "db_host=<RDS_ENDPOINT>" \
     -e "aws_region=us-east-1"
   ```

### Jenkins Pipeline Setup
```groovy
// Jenkinsfile-cd excerpt
stage('Deploy') {
    steps {
        withCredentials([aws(credentialsId: 'aws-creds', region: 'us-east-1')]) {
            ansiblePlaybook(
                playbook: 'deploy.yml',
                inventory: 'inventory.ini',
                extras: '-e "db_host=${RDS_ENDPOINT}"'
            )
        }
    }
}
```

## 🛡 Security Best Practices
- **SSH Hardening**
  ```yaml
  # roles/bootstrap/tasks/main.yml
  - name: Secure SSH config
    template:
      src: sshd_config.j2
      dest: /etc/ssh/sshd_config
    notify: restart sshd
  ```
  
- **IAM Least Privilege**
  ```yaml
  # roles/deploy/vars/main.yml
  aws_iam_role_policies:
    - arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
    - arn:aws:iam::aws:policy/AmazonRDSReadOnlyAccess
  ```

- **Encrypted Secrets**
  ```yaml
  # deploy.yml
  - name: Get DB credentials
    ansible.builtin.aws_ssm:
      name: "/prod/db/{{ item }}"
      region: "{{ aws_region }}"
    loop: ['user', 'pass']
    register: db_creds
  ```

## 📂 Project Structure
```
ansible-aws-config/
├── roles/
│   ├── bootstrap/         # System initialization tasks
│   │   ├── tasks/         # Package install, user setup
│   │   └── handlers/      # Service restarts
│   │
│   └── deploy/            # Application deployment
│       ├── tasks/         # Docker setup, app deployment
│       └── vars/          # Environment-specific configs
├── inventory.ini          # Target server definitions
├── deploy.yml             # Main playbook
└── Jenkinsfile-cd         # CI/CD pipeline definition
```

## 🌐 Related Projects
- [Terraform AWS Infrastructure](https://github.com/MariferVL/terraform-aws-infra)
- [WhatsNext API](https://github.com/MariferVL/whats-next-api)
- [DevOps Chronicles (Reference Implementation)](https://github.com/MariferVL/devops-chronicles-app)

## 🤝 Contribution Guidelines
1. Use Ansible best practices:
   - **YAML linting**: `ansible-lint`
   - **Modular design**: Break complex tasks into roles
   - **Idempotency**: All tasks must be rerun-safe

2. Update documentation with changes:
   ```bash
   # Generate role documentation
   ansible-doc -t role bootstrap
   ```

3. Test changes with:
   ```bash
   molecule test -s default
   ```

## 📜 License
Apache 2.0 License - See [LICENSE](LICENSE) for details
