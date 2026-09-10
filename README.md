# AWS EC2 Deployment: Java / Spring Boot

> Manual provisioning, securing, configuring, and deploying a Java/Spring Boot application on an AWS EC2 instance.

![AWS](https://img.shields.io/badge/AWS-EC2-orange)
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu-orange)
![Java](https://img.shields.io/badge/Java-17-blue)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-Application-green)
![Gradle](https://img.shields.io/badge/Build-Gradle-02303A)
![Linux](https://img.shields.io/badge/Linux-Server-FCC624)

---

## Table of Contents

* [Project Overview](#project-overview)
* [Objectives](#objectives)
* [Architecture](#architecture)
* [Technology Stack](#technology-stack)
* [Prerequisites](#prerequisites)
* [AWS Infrastructure](#aws-infrastructure)
* [Network and Security Configuration](#network-and-security-configuration)
* [Server Configuration](#server-configuration)
* [Application Build and Deployment](#application-build-and-deployment)
* [Linux User Management](#linux-user-management)
* [How to Reproduce This Deployment](#how-to-reproduce-this-deployment)
* [Implementation Evidence](#implementation-evidence)
* [Key Engineering Concepts](#key-engineering-concepts)
* [Engineering Lessons](#engineering-lessons)
* [Next Steps: From Manual to Production-Grade](#next-steps-from-manual-to-production-grade)
* [Project Outcome](#project-outcome)
* [Repository Structure](#repository-structure)
* [Conclusion](#conclusion)

---

## Project Overview

This project demonstrates the deployment of a Java/Spring Boot application onto a remote **AWS EC2 Linux server**, covering the full path from a locally built application artifact to a running, browser-verified application on cloud infrastructure.

The implementation combines:

* Cloud infrastructure provisioning
* Linux server administration
* SSH-based remote access
* Network security configuration
* Java runtime installation
* Application packaging with Gradle
* Secure artifact transfer
* Remote application execution
* Linux user and privilege management

The goal was not simply to spin up a virtual machine, but to understand the full set of infrastructure responsibilities involved in operating an application on a cloud server, end to end, done manually, before automating any of it.

> This deployment is intentionally manual. See [Next Steps](#next-steps-from-manual-to-production-grade) for how this would evolve toward a production setup.

---

## Objectives

1. Provision a Linux-based compute instance on AWS.
2. Configure controlled network access to the server.
3. Establish secure remote administration through SSH.
4. Prepare the server with the Java runtime required by the application.
5. Build and package the application locally.
6. Transfer the application artifact to the remote server.
7. Execute the application on the EC2 instance.
8. Expose the application through a controlled network port.
9. Manage Linux users and administrative privileges.
10. Validate the complete deployment from infrastructure to application layer.

---

## Architecture

```text
┌──────────────────────────────┐
│      Local Development       │
│                              │
│  Java / Spring Boot          │
│  Gradle Build                │
│  java-react-example.jar      │
└──────────────┬───────────────┘
               │
               │ SSH / SCP
               ▼
┌──────────────────────────────┐
│          AWS Cloud           │
│                              │
│  ┌────────────────────────┐  │
│  │      EC2 Instance      │  │
│  │                        │  │
│  │ Ubuntu Linux           │  │
│  │ Java 17                │  │
│  │ Spring Boot App        │  │
│  │ Port 7071              │  │
│  └────────────────────────┘  │
│                              │
│      Security Group          │
│      ├── SSH : 22            │
│      └── App : 7071          │
└──────────────┬───────────────┘
               │
               │ HTTP : 7071
               ▼
┌──────────────────────────────┐
│          Web Browser         │
│                              │
│  Application Verification    │
└──────────────────────────────┘
```

### Deployment Model

**Local environment**

* Application source code
* Gradle build process
* JAR artifact generation

**Cloud environment**

* Compute infrastructure
* Operating system
* Java runtime
* Network security
* Application execution

This separation reflects a fundamental cloud deployment model: the application is developed and packaged locally, while the runtime environment is hosted on remote infrastructure.

---

## Technology Stack

| Layer            | Technology               |
| ---------------- | ------------------------ |
| Cloud Provider   | AWS                      |
| Compute          | Amazon EC2                |
| Operating System | Ubuntu 26.04 LTS          |
| Architecture     | `x86_64`                  |
| Instance Type    | `t3.medium`                |
| Region           | `us-east-1`                |
| Storage          | 8 GB                       |
| Runtime          | OpenJDK 17                 |
| Application      | Java / Spring Boot         |
| Build Tool       | Gradle                     |
| Artifact         | `java-react-example.jar`   |
| Remote Access    | SSH                        |
| File Transfer    | SCP                         |
| Network Security | AWS Security Group          |
| Application Port | `7071`                       |

---

## Prerequisites

To reproduce this deployment yourself, you'll need:

* An AWS account with permissions to create EC2 instances and Security Groups
* A local machine with the AWS CLI or Console access
* Java 17 (JDK) and Gradle installed locally for building the artifact
* An SSH key pair (`.pem`) for connecting to the instance
* Basic familiarity with Linux command-line administration

---

## AWS Infrastructure

### EC2 Configuration

| Configuration    | Value            |
| ---------------- | ---------------- |
| Instance Type    | `t3.medium`      |
| Operating System | Ubuntu 26.04 LTS |
| Architecture     | `x86_64`         |
| Region           | `us-east-1`      |
| Storage          | 8 GB             |

![EC2 Instance](screenshots/01-ec2-instance.png)

The EC2 instance provides the compute layer on which the Linux operating system, Java runtime, and application execute.

---

## Network and Security Configuration

Network access was controlled using an AWS Security Group.

### Inbound Rules

| Port   | Protocol | Purpose            | Source        |
| ------ | -------- | ------------------ | ------------- |
| `22`   | TCP      | SSH administration | My IP address |
| `7071` | TCP      | Application access  | All sources   |

* SSH access was restricted to my IP address rather than exposed broadly.
* Port `7071` was opened for external application access.
* Outbound traffic remained under the default AWS configuration.

![Security Group Configuration](screenshots/02-security-group.png)

The Security Group acts as the network access boundary for the EC2 instance, controlling which traffic can reach services running on the server.

---

## Server Configuration

### Remote SSH Access

The EC2 server was accessed using the default Ubuntu account:

```text
ubuntu
```

```bash
ssh -i "<private-key>.pem" ubuntu@<ec2-public-dns>
```

> Sensitive credentials and private key material are intentionally excluded from this repository.

![SSH Connection](screenshots/03-ssh-connection.png)

### Java Runtime

The server was configured with **OpenJDK 17**.

```bash
sudo apt update
sudo apt install openjdk-17-jdk
java -version
```

![Java Installation](screenshots/04-java-installation.png)

---

## Application Build and Deployment

### Build

The application is a Java/Spring Boot application, built locally with Gradle:

```bash
./gradlew build
```

This produces the deployable artifact: `build/libs/java-react-example.jar`

![Gradle Build](screenshots/05-gradle-build.png)

### Artifact Transfer

The JAR was transferred to the EC2 server via SCP:

```bash
scp -i <private-key>.pem build/libs/java-react-example.jar ubuntu@<ec2-public-ip>:
```

![JAR Transfer](screenshots/06-jar-transfer.png)

### Execution

The application was started on the instance:

```bash
java -jar java-react-example.jar
```

The application runs on port `7071`.

![Application Running](screenshots/07-application-running.png)

### Verification

```text
http://<ec2-public-ip>:7071/
```

Successful browser access confirmed end-to-end connectivity between the external client and the application running on EC2.

![Application in Browser](screenshots/08-application-browser.png)

---

## Linux User Management

A dedicated Linux user was created for server administration, rather than relying solely on the default `ubuntu` account:

```bash
sudo adduser pierre
sudo usermod -aG sudo pierre
```

This provided hands-on practice with Linux identity and privilege management.

![Linux User Management](screenshots/09-linux-user.png)

---

## How to Reproduce This Deployment

1. Launch a `t3.medium` Ubuntu EC2 instance in your preferred region.
2. Create a Security Group allowing SSH (port 22, your IP only) and your app port (e.g. 7071, open or restricted as needed).
3. Connect via SSH using your key pair.
4. Install Java 17 (`sudo apt update && sudo apt install openjdk-17-jdk`).
5. Build the application locally with `./gradlew build`.
6. Transfer the JAR to the instance with `scp`.
7. Run the JAR on the instance with `java -jar <artifact>.jar`.
8. Verify in a browser at `http://<ec2-public-ip>:<port>/`.
9. (Optional) Create a dedicated non-root Linux user with sudo privileges for ongoing administration.

---

## Implementation Evidence

| Evidence                 | Demonstrates                      |
| ------------------------ | ---------------------------------- |
| EC2 instance              | Cloud compute provisioning         |
| Security Group            | Network access control             |
| SSH connection            | Remote server administration       |
| Java installation         | Runtime preparation                |
| Gradle build               | Application packaging              |
| JAR transfer               | Remote artifact deployment         |
| Running application        | Application execution              |
| Browser verification       | End-to-end connectivity            |
| Linux user configuration   | Identity and privilege management  |

---

## Key Engineering Concepts

**Infrastructure as a Service:** compute infrastructure is provisioned and managed remotely rather than running the application entirely on a local workstation.

**Remote Server Administration:** SSH provides secure command-line access to configure and manage the remote Linux environment from a local machine.

**Network Boundaries:** the AWS Security Group separates administrative access (SSH) from application access (port `7071`), reflecting the different trust and access requirements of each.

**Artifact-Based Deployment:** source code is compiled into a JAR artifact via Gradle, then transferred and executed on the target environment:

```text
Source Code → Build → Artifact → Transfer → Runtime
```

This is the conceptual foundation for CI/CD and automated deployment pipelines.

**Linux Privilege Management:** creating a dedicated non-default user with scoped sudo access reflects standard operating-system-level identity and access management practice.

---

## Engineering Lessons

**1. Infrastructure and application layers are different.**
A working application depends on compute resources, OS, runtime dependencies, network access, security controls, and user privileges, not just code. The EC2 instance becomes part of the application's runtime environment, not just a remote machine.

**2. Network configuration is part of deployment.**
An application can run correctly and still be unreachable if the network layer doesn't permit the required traffic. Success required alignment between the app, its port, the Security Group, and client connectivity.

**3. Deployment requires an artifact, not source code.**
Gradle produced a JAR that was transferred and executed remotely: the deployment unit, not the raw source. This is foundational to understanding artifact repositories and automated pipelines.

**4. Server access requires deliberate security controls.**
SSH is a security-sensitive boundary. Restricting it to a known IP materially reduces exposure compared to open access.

---

## Next Steps: From Manual to Production-Grade

This deployment was intentionally manual, to build first-hand understanding of each layer. A production version of this setup would add:

* **Process management:** run the app as a `systemd` service instead of a foreground `java -jar` process, so it survives reboots and crashes.
* **Reverse proxy + TLS:** put Nginx in front of the app and serve over HTTPS instead of exposing the JVM directly on `7071`.
* **Static addressing:** attach an Elastic IP so the public address doesn't change on instance restart.
* **Secrets management:** move credentials and config out of the shell/environment and into AWS Secrets Manager or SSM Parameter Store.
* **Infrastructure as Code:** provision the EC2 instance and Security Group with Terraform or CloudFormation instead of the console.
* **CI/CD:** automate build, artifact, deploy with GitHub Actions instead of manual `scp`.
* **Least-privilege runtime:** run the JVM as a non-root, application-specific Linux user rather than under an admin-capable account.
* **Monitoring:** add basic health checks and logging (CloudWatch or equivalent).

---

## Project Outcome

* Provisioned AWS EC2 compute environment
* Configured Ubuntu Linux server
* Restricted SSH access
* Installed and verified Java 17 runtime
* Packaged application locally with Gradle
* Transferred JAR artifact to the cloud server
* Executed the Spring Boot application remotely
* Verified application via browser
* Exposed application through port `7071`
* Created a dedicated Linux user with sudo privileges

The result is a complete, manually executed cloud deployment workflow from **local application build to remotely accessible application**.

---

## Repository Structure

```text
AWS-Server-Deployment/
│
├── README.md
│
├── java-react-app/
│   ├── build.gradle
│   ├── gradle/
│   └── build/
│
├── src/
│   ├── main/
│   └── test/
│
├── screenshots/
│   ├── 01-ec2-instance.png
│   ├── 02-security-group.png
│   ├── 03-ssh-connection.png
│   ├── 04-java-installation.png
│   ├── 05-gradle-build.png
│   ├── 06-jar-transfer.png
│   ├── 07-application-running.png
│   ├── 08-application-browser.png
│   └── 09-linux-user.png
│
├── .gitignore
│
└── ...
```

---

## Conclusion

This project demonstrates the practical operation of a Java application on cloud infrastructure, spanning the infrastructure, OS, networking, runtime, application, and access-management layers required to move an app from a developer workstation to a remotely accessible cloud environment.

It's a deliberate first step: manual, transparent, and fully understood, before layering in automation, infrastructure as code, and production hardening in future projects.