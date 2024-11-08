# SRE Fundamentals with Google

## Week 2 Project

### Implementing Canary Deployments at UpCommerce: A Journey in Risk Mitigation

You have just been appointed as the new SRE team lead at UpCommerce. And on your first day as team lead, you are handed a complaint brief: UpCommerce.com, your company’s e-commerce platform, which serves over 10,000 merchants across North America, has been experiencing increased latency and stability issues during peak shopping hours. In the last standup meeting before your appointment, the development team proposed a significant architectural change to their monolithic application, starting with the user interface layer. In that meeting, James Martinez, the Engineering Lead, shared some concerning metrics:

* Customer complaints about slow page loads have increased by 40% in the last quarter
* The platform experienced three major outages during last year's Black Friday sale
* Rollbacks of problematic deployments took an average of 45 minutes, resulting in significant revenue loss for merchants

Your SRE team has proposed modernizing UpCommerce's deployment strategy using a canary release approach. This would allow them to safely test new versions of the application with a small subset of users before rolling out changes platform-wide. However, the stakes are high - UpCommerce's merchants process an average of $2.5 million in daily transactions, and any disruption to service directly impacts their livelihoods.

The SRE team's proposal includes:

* Implementing a canary deployment strategy that routes 20% of traffic to the new version
* Setting up comprehensive monitoring and alerting to quickly detect any issues
* Establishing clear success metrics and automatic rollback triggers
* Using Helm for managing these deployments, as it's already part of UpCommerce's toolchain

James has approved a pilot project to test this approach. "We need to move fast," he explains, "but we can't compromise on reliability. Our merchants depend on us for their livelihood." He's given your team two weeks to demonstrate a proof of concept using a simplified version of the UpCommerce front end.

### Project Objective

This project will walk you through implementing this canary deployment strategy, setting up the necessary monitoring, and establishing the automation required for safe, reliable deployments. You'll learn how to use modern DevOps tools and SRE practices to solve real-world challenges in a high-stakes e-commerce environment.

### Getting your development environment ready

For this week's project, you will use GitHub Codespaces as your development environment.

#### Steps

1. Create a fork of the week's repository.
1. When you have created a fork of the week's repository, start a codespace on the main branch. To ensure that you do not run out of system resource, choose the 4-core, 16GB RAM machine type (see image below)
1. Run the command below in your codespace's terminal to create a single-node, Kubernetes cluster using Minikube: 
    * `minikube start`
1. Once your Minikube cluster is running, enter the command below to enable the Ingress add-on in your Minikube cluster:
1. minikube addons enable ingress
1. Create a namespace using the following command:
    * `kubectl create namespace canary-demo`
  This creates a namespace in your Kubernetes cluster named canary-demo. It is within this namespace that you'll do all the tasks required for this project.

### Project Code Structure

#### The UpCommerce Application

At the core of this project is a lightweight Flask application representing a simplified version of UpCommerce's front end. The application serves an HTML page with a configurable background color - white for the stable version (v1) and green for the canary version (v2). This visual distinction makes it easy to verify our canary deployment in action. The application is also instrumented with Prometheus metrics, exposing key data points through its /metrics endpoint. These metrics include request counts by version, allowing us to monitor traffic distribution between the stable and canary deployments. While this is a simplified version of UpCommerce's actual front end, it effectively demonstrates the principles of canary deployments and monitoring.

#### Docker Configuration

Pre-built Docker images (uonyeka/canary-demo:v1 and uonyeka/canary-demo:v2) are available on Docker Hub for immediate use in this project. These pre-built docker images are readily usable on Windows and Linux (which is the operating system that GitHub Codespaces run on). However, for those interested in building their own images or running on different architectures, we provide the complete Dockerfile and source code. The Dockerfile uses Python 3.9-slim as the base image and includes all necessary dependencies specified in requirements.txt. The build process is straightforward: it copies the application code, installs dependencies, and sets up environment variables for version identification. For custom builds, you'll need to modify the image repository and tags in your values.yaml file accordingly.

#### Helm Chart Structure

The canary-demo Helm chart is structured to manage both the stable and canary deployments of our application. In the templates directory, you'll find carefully crafted Kubernetes manifests: deployment.yaml handles both the main and canary deployments, service.yaml creates the necessary service endpoints, ingress.yaml manages traffic routing with appropriate canary annotations, configmap.yaml manages environment-specific configurations, and servicemonitor.yaml sets up Prometheus monitoring. The values.yaml file serves as the central configuration hub, allowing easy customization of image versions, replica counts, resource limits, and canary deployment settings. This structure follows Helm best practices while keeping complexity manageable for educational purposes.

### Unit Testing

The project includes a comprehensive test suite in ./tests/test_app.py that validates both functional and monitoring aspects of our canary deployment. The tests use Python's unittest framework and the requests library to verify the application's behavior. Key test cases include verifying the correct response from both main and canary deployments (checking for version-specific content and background colors), ensuring proper metric exposition at the /metrics endpoint, and validating the expected traffic distribution between versions. The test suite is designed to run against the deployed application, making HTTP requests with appropriate host headers to simulate real-world traffic. A particularly interesting test case checks for the canary deployment by making multiple requests and ensuring that approximately 20% of responses come from the canary version, validating our traffic-splitting configuration.

### Required Tasks

1. You are to create a deployment of both the original and canary versions of upcommerce.com in your Minikube cluster. Client requests should be served in the ratio 80:20 (80% of the requests are served from the main deployment while 20% is to be served from the canary).
1. You are also to setup monitoring using the prometheus-grafana helm chart provided in the canary-demo template files. 
1. You have been notified by the front-end team that an updated UI is available in the company’s Docker Hub account for canary deployment. The Docker image for this update can be found in the repository on the Docker Hub (uonyeka/canary-demo:v3). Your task here is to pull this update and deploy the v3 canary. 
1. Due to customer complaints about the UI background colors not being easy on the eyes, you’ll need to roll back to the previous version of the canary (v2).
1. The project repo contains a YAML file named answers.yml. In this file, you’ll see a set of key-value pairs like the one below:  
  ```
pods_status:
  main_pods_running: #TODO: Enter number of main pods running
  canary_pods_running: #TODO: Enter number of canary pods runningY
  ```

Please make sure to fill in each of the spaces marked #TODO with the values obtained from your running deployment. For instance, to retrieve the answers to the questions about pod status mentioned above, you will need to run the command kubectl get pods -n canary-demo in your terminal. This command will provide you with the number of pods running for both the main deployment and the canary deployment.

When you are done, push your changes to your fork of the GitHub repository and paste the link to your repo on the project submission page.
