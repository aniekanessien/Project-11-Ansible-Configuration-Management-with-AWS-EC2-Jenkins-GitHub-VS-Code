# Project 11 — Ansible Configuration Management on AWS

### Jenkins • GitHub Webhooks • Linux • SSH • VS Code

## Overview

I built a centralized configuration-management environment on AWS using Ansible, Jenkins, GitHub, and six Linux EC2 instances.

The environment consists of a Jenkins–Ansible control server and five managed servers: two web servers, a database server, an NFS server, and an Ubuntu load-balancer server. I organized the targets into inventory groups and used a common playbook to configure packages across RHEL-family and Ubuntu hosts.

I then connected the GitHub repository to Jenkins through a push webhook and tested the build workflow, including Ansible connectivity and deployment-related console output.

The implementation involved troubleshooting as well as successful execution. I documented a Jenkins disk-space issue, an initial unsuccessful build, an Ansible connectivity failure, and additional Jenkins console errors. Keeping those records helped me explain how I investigated the environment and validated the working result.

## Project at a Glance

| Area | Implementation |
|---|---|
| Cloud platform | Amazon Web Services |
| Infrastructure | Six EC2 instances |
| Control server | Jenkins and Ansible on Ubuntu |
| Managed servers | Web1, Web2, database, NFS, and load balancer |
| Managed operating systems | RHEL-family Linux and Ubuntu |
| Configuration management | Ansible inventory and YAML playbooks |
| Configuration exercise | Wireshark package installation |
| Version control | Git and GitHub |
| Jenkins workflow | Repository-connected builds and webhook integration |
| Remote management | SSH over the AWS private network |
| Editor | Visual Studio Code |
| Evidence | 46 implementation and troubleshooting screenshots |

The server names identify their intended infrastructure roles. The common package playbook does not, by itself, deploy a database application, configure NFS exports, or configure an application load balancer.

## Why I Built This

Manually configuring multiple servers creates repeated work and makes configuration differences harder to track. I wanted to manage several machines from one control node and keep the required configuration in version control.

My objectives were to:

- Establish a working Ansible control node.
- Connect to managed servers through their private addresses.
- Group hosts by infrastructure role.
- Apply configuration tasks across different Linux distributions.
- Validate connectivity before running package tasks.
- Verify the configuration on a managed server.
- Connect repository updates to Jenkins through GitHub webhooks.
- Diagnose failed attempts and document the recovery evidence.

## Architecture

### Infrastructure and Configuration Management

```mermaid
flowchart TD
    developer["Developer laptop and VS Code"]
    github["GitHub repository"]
    control["EC2: Jenkins and Ansible control node"]
    web["Web1 and Web2: RHEL-family Linux"]
    database["Database: RHEL-family Linux"]
    nfs["NFS: RHEL-family Linux"]
    lb["Load balancer: Ubuntu"]

    developer -->|"Commit and push"| github
    github -->|"Push webhook"| control
    control -->|"Ansible over SSH"| web
    control -->|"Ansible over SSH"| database
    control -->|"Ansible over SSH"| nfs
    control -->|"Ansible over SSH"| lb
```






The load-balancer server is another Ansible-managed target. It is not downstream of the database or NFS server in the configuration-management path.

### Component Responsibilities

| Component | Responsibility |
|---|---|
| Developer laptop | Editing, Git operations, and administration |
| VS Code | Editing inventory, playbooks, and documentation |
| GitHub | Storing project changes and sending repository-event notifications |
| Jenkins | Checking out the repository and executing the configured job |
| Ansible control node | Reading inventory and executing configuration tasks |
| Managed nodes | Receiving tasks through SSH and applying the requested changes |

Jenkins and Ansible share a server in this lab, but they do not necessarily share the same runtime identity. Jenkins jobs run under their configured execution account, which needs its own access to credentials, files, and managed hosts.

### Repository-to-Build Workflow

```mermaid
sequenceDiagram
    participant Developer
    participant GitHub
    participant Jenkins
    participant Servers as Managed servers

    Developer->>GitHub: Push project changes
    GitHub->>Jenkins: Send push webhook
    Jenkins->>GitHub: Check out configured branch
    Jenkins->>Jenkins: Execute build steps
    opt Ansible step configured in job
        Jenkins->>Servers: Execute connectivity or playbook tasks
        Servers-->>Jenkins: Return host and task results
    end
    Jenkins->>Jenkins: Record console output and artifacts
```

A delivered webhook, a successful checkout, and a successful Ansible run are separate checkpoints. The build console identifies which commands ran and what each managed host returned.

## Tools and Resources

| Tool or resource | Purpose |
|---|---|
| Amazon EC2 | Hosting the control node and managed servers |
| AWS VPC networking | Connecting instances through their private addresses |
| AWS Security Groups | Controlling permitted network access |
| Ubuntu Linux | Control-node and load-balancer operating system family |
| RHEL-family Linux | Operating system family for the other managed targets |
| Java runtime | Supporting Jenkins |
| Jenkins | Build orchestration and execution records |
| Jenkins Git/GitHub integration | Repository checkout and webhook-triggered builds |
| Ansible | Centralized configuration management |
| YAML | Structured inventory and playbook definitions |
| Git | Tracking local changes |
| GitHub | Hosting the repository and configuring webhooks |
| SSH | Remote administration and Ansible transport |
| ssh-agent and ssh-add | Making an SSH identity available to a shell session |
| yum/dnf-compatible package management | Package configuration on RHEL-family hosts |
| apt | Package configuration on Ubuntu |
| Wireshark | Package used for the configuration-management exercise |
| systemctl | Checking service state |
| Linux storage tools | Investigating the Jenkins storage issue |
| VS Code | Editing and organizing the project |
| Browser | Accessing Jenkins and GitHub |
| Local laptop | Providing the development and administration environment |

Exact package versions, instance sizes, and the AWS Region should be taken from the deployed environment rather than assumed from the training examples.

## Repository Organization

| Path | Purpose |
|---|---|
| `README.md` | Architecture, implementation, and troubleshooting |
| `inventory/` | Managed-host definitions and connection variables |
| `playbooks/` | Configuration tasks |
| `screenshots/` | Installation, execution, and recovery evidence |

The examples in this documentation use `inventory/dev.yml` and `playbooks/common.yml`.

## Implementation

### 1. Preparing AWS Infrastructure

I prepared the control server and managed EC2 instances, then configured the network access needed for administration.

The intended management design uses private addresses for Ansible connections. Managed-server SSH access is restricted through a security-group rule referencing the control-server security group.

![AWS EC2 instances](screenshots/01-aws-ec2-instances.png)

![Control-node security group](screenshots/02-control-node-security-group.png)

The manual also permits public HTTP access on the shared managed-server group. Therefore, restricting SSH does not mean every port on those machines is private.

### 2. Installing and Accessing Jenkins

I connected to the Jenkins server, completed installation and initial setup, prepared plugins, and accessed the dashboard.

![Jenkins server SSH access](screenshots/03-jenkins-server-ssh-access.png)

![Jenkins installation](screenshots/04-jenkins-installation.png)

![Jenkins plugin setup](screenshots/05-jenkins-plugin-setup.png)

![Jenkins dashboard](screenshots/06-jenkins-dashboard.png)

This established the build platform before I configured the repository integration.

### 3. Preparing the Ansible Control Node

I installed Ansible and Git and checked that the commands were available.

![Ansible version check](screenshots/07-ansible-version-check.png)

![Git version check](screenshots/08-git-version-check.png)

Ansible provides the configuration-management layer. Git provides the version-controlled project source used by the development and Jenkins workflows.

### 4. Establishing SSH Connectivity

I prepared the SSH identity and tested connections from the control node to managed servers.

![SSH-agent identity](screenshots/09-ssh-agent-identity.png)

![Control-node access to Web1](screenshots/10-control-node-web1-ssh-access.png)

![Control-node access to Web2](screenshots/11-control-node-web2-ssh-access.png)

![Control-node access to NFS](screenshots/12-control-node-nfs-ssh-access.png)

This checkpoint separated connection problems from playbook problems. Before investigating a configuration task, I needed to establish that the target was reachable with the intended username and identity.

The screenshots describe access without repeatedly entering the key path. This still uses SSH authentication; it does not imply that authentication was disabled.

### 5. Organizing the Inventory

I grouped the managed machines by role so that playbooks could target the appropriate servers.

| Group | Targets |
|---|---|
| `webservers` | Web1 and Web2 |
| `db` | Database server |
| `nfs` | NFS server |
| `lb` | Ubuntu load balancer |

![Ansible inventory structure](screenshots/13-ansible-inventory-structure.png)

The training manual uses an inventory filename ending in `.yml` but shows INI-style content. The following is a correctly structured YAML equivalent, with placeholders replacing real addresses.

```yaml
---
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: WEB1_PRIVATE_IP
        web2:
          ansible_host: WEB2_PRIVATE_IP
      vars:
        ansible_user: ec2-user

    db:
      hosts:
        database:
          ansible_host: DATABASE_PRIVATE_IP
      vars:
        ansible_user: ec2-user

    nfs:
      hosts:
        nfs_server:
          ansible_host: NFS_PRIVATE_IP
      vars:
        ansible_user: ec2-user

    lb:
      hosts:
        load_balancer:
          ansible_host: LOADBALANCER_PRIVATE_IP
      vars:
        ansible_user: ubuntu
```

This is a sanitized reference example, not an export containing actual infrastructure addresses.

The SSH user must match the chosen image. The manual uses `ec2-user` for its RHEL-family examples and `ubuntu` for Ubuntu.

Useful inventory checks are:

```bash
ansible-inventory -i inventory/dev.yml --list
ansible-inventory -i inventory/dev.yml --graph
```

### 6. Validating Ansible Connectivity

I ran Ansible's ping module before executing the common configuration playbook.

```bash
ansible all \
  -i inventory/dev.yml \
  -m ansible.builtin.ping
```

![Successful Ansible connectivity](screenshots/14-ansible-connectivity-success.png)

Ansible's ping module is not an ICMP network ping. It checks the management connection and whether the module can run in a usable remote Python environment.

A `pong` response establishes this connectivity checkpoint. It does not independently validate package repositories or privilege escalation.

### 7. Creating the Common Playbook

I used separate plays for the RHEL-family servers and the Ubuntu load balancer so that package configuration followed the target operating system.

![Common playbook created](screenshots/15-common-playbook-created.png)

![Common playbook content](screenshots/19-common-playbook-content.png)

The manual's configuration pattern is shown below. It is a reference example; the appropriate yum/dnf module depends on the target distribution and installed Ansible version.

```yaml
---
- name: Configure RHEL-family servers
  hosts:
    - webservers
    - nfs
    - db
  become: true

  tasks:
    - name: Install or update Wireshark
      yum:
        name: wireshark
        state: latest

- name: Configure the Ubuntu load balancer
  hosts: lb
  become: true

  tasks:
    - name: Refresh apt package metadata
      apt:
        update_cache: true

    - name: Install or update Wireshark
      apt:
        name: wireshark
        state: latest
```

`become: true` allows the package tasks to use elevated privileges.

`state: latest` requests the latest package available from the configured repositories at the time of execution. It does not pin a reproducible package version.

### 8. Executing and Verifying Configuration

I executed the common playbook against the inventory and checked the results.

```bash
ansible-playbook \
  -i inventory/dev.yml \
  playbooks/common.yml
```

![Successful common-playbook execution](screenshots/16-common-playbook-execution-success.png)

![Package verification on Web1](screenshots/17-web1-package-verification.png)

I treated the managed-host recap as part of the result. A completed command needs to be assessed for failed or unreachable hosts, not only its final console line.

The package-verification screenshot provides a separate check on a managed machine.

Useful package-query examples are:

```bash
rpm -q wireshark
```

For an Ubuntu target:

```bash
dpkg-query -W wireshark
```

These commands are diagnostic references rather than additional results claimed in the screenshots.

### 9. Working Through VS Code

I documented access to the Ansible controller through the VS Code workflow.

![VS Code access to the control node](screenshots/18-vscode-control-node-access.png)

This connected project editing with the environment where Ansible commands were executed.

### 10. Preparing Jenkins-Specific Access

After validating the interactive workflow, I prepared the Jenkins execution context.

![Jenkins service running](screenshots/20-jenkins-service-running.png)

![Jenkins known-hosts setup](screenshots/21-jenkins-known-hosts-setup.png)

![Jenkins access to managed hosts](screenshots/22-jenkins-managed-host-ssh-access.png)

![Jenkins credential setup — sensitive values redacted](screenshots/23-jenkins-credential-setup.png)

Host-key trust and client authentication have different responsibilities. Known-host entries identify the remote server; the SSH identity authenticates the client.

A key available to my interactive shell is not automatically available to Jenkins. The job's execution account needs the appropriate access independently.

### 11. Validating Jenkins Builds

I connected the Jenkins job to the repository and retained the successful build and console records after earlier attempts.

The manual's job design uses a Freestyle project, Git source control, the `main` branch, a GitHub hook trigger, and artifact archiving.

![Successful Jenkins node build](screenshots/24-jenkins-node-build-success.png)

![Build 4 console success](screenshots/25-jenkins-build-04-console-success.png)

![Successful build 5](screenshots/26-jenkins-build-05-success.png)

![Build 5 console success](screenshots/27-jenkins-build-05-console-success.png)

A successful build is assessed against its configured steps. Checkout and archiving alone do not execute an Ansible playbook.

### 12. Connecting GitHub Webhooks

I configured GitHub to notify Jenkins when a repository push occurred.

![GitHub webhook configuration](screenshots/28-github-webhook-configuration.png)

The lab endpoint follows this pattern:

```text
http://JENKINS_PUBLIC_IP:8080/github-webhook/
```

The manual uses a JSON payload and push events.

I then changed the repository documentation and pushed the update to test the event flow.

![README change for trigger testing](screenshots/29-readme-change-for-trigger-test.png)

![Git push trigger test](screenshots/30-git-push-trigger-test.png)

![Webhook-triggered builds](screenshots/31-webhook-triggered-builds.png)

### 13. Reviewing Ansible and Webhook Build Results

I retained the Jenkins-side Ansible connectivity and deployment-related console evidence, then checked webhook activity on both platforms.

![Jenkins Ansible connectivity console](screenshots/32-jenkins-ansible-connectivity-console.png)

![Jenkins Ansible connectivity success](screenshots/33-jenkins-ansible-connectivity-success.png)

![Jenkins Ansible deployment console](screenshots/34-jenkins-ansible-deployment-console.png)

![Successful webhook-triggered build 7](screenshots/35-jenkins-webhook-build-07-success.png)

![Jenkins webhook activity](screenshots/36-jenkins-webhook-activity.png)

![Successful GitHub webhook delivery](screenshots/37-github-webhook-delivery-success.png)

![Final commit and Git status](screenshots/38-final-git-commit-status.png)

These checks cover different parts of the workflow:

| Check | What it establishes |
|---|---|
| GitHub delivery record | Whether GitHub delivered the event |
| Jenkins webhook activity | Whether Jenkins received and processed webhook activity |
| Build record | Whether the intended job ran |
| Console output | Which commands executed and their results |
| Ansible recap | What happened on each target host |

The exact deployment commands and affected hosts should be read from the build console rather than inferred from the build colour alone.

## Troubleshooting and Recovery

### Issue 1 — Jenkins Disk-Space Warning

I encountered a Jenkins disk-space issue and retained the warning before making the storage correction.

![Jenkins disk-space error](screenshots/39-error-jenkins-disk-space.png)

My evidence records a capacity change described as an increase from 2 GB to 20 GB, followed by an updated block-device inspection.

![Storage-capacity correction](screenshots/40-storage-capacity-increased.png)

![Updated block-device capacity](screenshots/41-updated-block-device-capacity.png)

The photos show the filenames rather than readable storage output. They do not establish which filesystem was affected or the exact resize command, so I have not reconstructed those details.

#### Investigation Approach

For this class of failure, I distinguish:

- The block device's size.
- The mounted filesystem's available space.
- Inode availability.
- Temporary-storage capacity.
- Jenkins node resource thresholds.

Useful diagnostic commands include:

```bash
df -h
df -i
lsblk
findmnt /tmp
```

Jenkins can mark a node unavailable when monitored resources fall outside configured thresholds. The service can therefore remain running while builds cannot use the node.

#### What I Learned

Increasing a device's capacity and making that capacity available to the relevant filesystem are separate checks. I need to recheck the storage Jenkins actually uses and then validate node availability.

The subsequent successful build evidence records progress after the earlier problem without assigning every failed build to the same cause.

### Issue 2 — Initial Jenkins Build Failure

I retained the unsuccessful initial build or node-execution attempt.

![Initial Jenkins build failure](screenshots/42-error-initial-jenkins-build.png)

#### Investigation Approach

I separate a job waiting for an executor from a job that begins execution and fails.

The relevant checks include:

1. Node online/offline state.
2. Executor availability.
3. Resource-monitor warnings.
4. Repository checkout.
5. The first failing command in the console output.

Later successful builds are documented in screenshots 24–27.

#### What I Learned

A failed or queued build is a symptom. The node state and console output determine whether the problem is capacity, scheduling, source checkout, credentials, or a build command.

### Issue 3 — Ansible Connectivity Failure

I recorded an unsuccessful ping attempt and the successful follow-up connectivity evidence.

![Ansible connectivity failure](screenshots/43-error-ansible-connectivity.png)

#### Investigation Approach

I start with inventory parsing, then test the same address, username, and identity that Ansible uses.

```bash
ansible-inventory \
  -i inventory/dev.yml \
  --graph
```

For detailed connection output:

```bash
ansible all \
  -i inventory/dev.yml \
  -m ansible.builtin.ping \
  -vvv
```

Relevant checks include the target private address, SSH user, credentials, network rules, host-key trust, and remote Python environment.

The exact error text is not readable in the supplied photograph. These are investigation areas, not an invented confirmed root cause.

The successful connectivity checkpoint appears in screenshot 14, with Jenkins-side connectivity evidence in screenshots 32–33.

#### What I Learned

I should establish connectivity before investigating configuration tasks. Otherwise, I risk changing a playbook when the task never reached the target.

### Issue 4 — Jenkins Console and Job Errors

I retained additional console and job error screenshots alongside the later successful execution records.

![Jenkins console error](screenshots/44-error-jenkins-console-output.png)

![Jenkins job error](screenshots/45-error-jenkins-job.png)

#### Investigation Approach

I identify the first failed stage or command and compare its execution context with the successful interactive test.

Important differences can include:

| Area | Question |
|---|---|
| User | Which account executes the job? |
| Credentials | Can that account access the intended SSH identity? |
| Working directory | Is the command running from the repository workspace? |
| Paths | Can it find the inventory and playbook? |
| Environment | Are the required commands available? |
| Permissions | Can it read files and elevate privileges where required? |

These checks describe the troubleshooting method without assigning an unreadable console message to a specific cause.

#### What I Learned

Success in my shell does not guarantee success in Jenkins. The failing command needs to be understood in the job's actual runtime environment.

## Evidence and Validation Summary

| Stage | Screenshots |
|---|---|
| Infrastructure and security rules | 01–02 |
| Jenkins installation and dashboard | 03–06 |
| Ansible and Git versions | 07–08 |
| SSH identity and managed-host access | 09–12 |
| Inventory and connectivity | 13–14 |
| Playbook creation and execution | 15–16, 19 |
| Managed-host package verification | 17 |
| VS Code access | 18 |
| Jenkins service and credentials | 20–23 |
| Successful builds and console output | 24–27 |
| Webhook configuration and trigger test | 28–31 |
| Jenkins-side Ansible evidence | 32–34 |
| Build and webhook results | 35–37 |
| Git status | 38 |
| Storage warning and correction | 39–41 |
| Build and connectivity failures | 42–45 |
| Documentation | 46 |

![Project README documentation](screenshots/46-project-readme-documentation.png)

## Results and Scope

The project demonstrates centralized package configuration, managed-host connectivity, Jenkins build integration, and GitHub push-webhook testing.

The implementation evidence includes:

- A six-instance management environment.
- Ansible inventory organization.
- Successful connectivity checks.
- Common-playbook execution.
- Package verification on a managed server.
- Jenkins-specific SSH preparation.
- Successful build and console records.
- GitHub and Jenkins webhook activity.
- Troubleshooting evidence and subsequent working checkpoints.

The server roles provide the targets for configuration management. This project does not claim that the common package playbook deploys the full web, database, NFS, or load-balancing application stack.

I also do not claim a measured deployment-time reduction, production SLA, or proven zero-change second run without corresponding test evidence.

## Security and Operational Considerations

The managed-host SSH design limits the management path to the control server. The manual's public HTTP rule remains a separate allowance.

For a hardened environment, I would protect Jenkins with HTTPS, restrict administrative access, configure webhook-secret verification, and manage deployment credentials through an approved Jenkins credential workflow.

The combined Jenkins–Ansible server is convenient for a lab but remains a single management dependency. Production work would also consider dedicated agents, separation of responsibilities, and recovery procedures.

Private keys, passwords, tokens, and AWS credentials are excluded from public documentation. Credential screenshots must be redacted before publication.

## What I Learned

This project helped me understand configuration management as a sequence of verifiable steps. The inventory identifies the targets, the playbook describes the requested state, and the execution results show what happened on each host.

I learned to separate the developer environment from the Jenkins environment. Credentials, paths, users, and permissions must be valid for the account that actually executes the automation.

The failures reinforced the value of checkpoints. I could investigate infrastructure access, inventory parsing, Ansible connectivity, package tasks, Jenkins execution, and webhook delivery independently.

I also learned that good documentation connects a result to evidence. A green build is useful, but the console commands and managed-host recap explain what the build accomplished.

## Skills Demonstrated

| Skill | Application in this project |
|---|---|
| Linux administration | SSH, services, packages, and storage investigation |
| AWS networking | Private-address management and security-group rules |
| Ansible | Inventory, playbooks, privilege escalation, and connectivity checks |
| Cross-platform configuration | RHEL-family and Ubuntu package tasks |
| Jenkins | Build configuration, console inspection, and troubleshooting |
| GitHub integration | Push webhooks and delivery checks |
| Git | Version-controlled updates and commits |
| Technical documentation | Architecture, evidence mapping, and failure analysis |

## Future Improvements

My next improvements would be to:

- Add a Jenkinsfile so job behaviour is version controlled.
- Move repeated configuration into Ansible roles.
- Add inventory validation and playbook linting.
- Record repeat execution to evaluate idempotent behaviour.
- Pin package versions where reproducibility is required.
- Use managed credentials and encrypted secret handling.
- Configure HTTPS and webhook-secret verification.
- Archive only useful artifacts and define build retention.
- Separate build agents from the control service.
- Provision infrastructure with Terraform.
- Add monitoring and configuration-drift checks.

These are planned improvements rather than completed features.

## References

- [Ansible documentation](https://docs.ansible.com/)
- [Ansible ping module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/ping_module.html)
- [Ansible YAML inventory](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/yaml_inventory.html)
- [Jenkins documentation](https://www.jenkins.io/doc/)
- [Jenkins node management](https://www.jenkins.io/doc/book/managing/nodes/)
- [Jenkins GitHub plugin](https://plugins.jenkins.io/github)
- [Amazon EC2 documentation](https://docs.aws.amazon.com/ec2/)