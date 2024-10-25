# Docker Image Scanning with Podman and Trivy (No-Sudo Environment)

This GitHub repository contains a GitHub Actions workflow for scanning Docker images using Trivy with Podman in a rootless (no-sudo) environment. This setup is ideal for environments where sudo access is restricted and Podman is used as a Docker alternative.

Workflow Overview
The workflow is designed to:

Check if Podman is installed.
Install Trivy locally without requiring sudo.
Pull a sample Docker image (e.g., alpine:latest) using Podman.
Scan the Docker image for vulnerabilities using Trivy.
Workflow Trigger
This workflow can be triggered manually via workflow_dispatch.
#####
Workflow YAML
yaml
Copy code
```
name: Docker Image Scanning with Podman and Trivy (No-Sudo Environment)

on:
  workflow_dispatch:

jobs:
  scan_image:
    runs-on: self-hosted

    steps:
      # Step 1: Check if Podman is installed
      - name: Check if Podman is installed
        run: |
          if command -v podman > /dev/null; then
            echo "Podman is installed"
            podman --version
          else
            echo "Podman is not installed"
            exit 1
          fi

      # Step 2: Install Trivy Locally without sudo
      - name: Install Trivy Locally
        run: |
          mkdir -p $HOME/bin
          export PATH=$PATH:$HOME/bin
          curl -L https://github.com/aquasecurity/trivy/releases/download/v0.56.1/trivy_0.56.1_Linux-64bit.tar.gz -o trivy.tar.gz
          tar zxvf trivy.tar.gz
          mv trivy $HOME/bin/
          rm trivy.tar.gz

      # Step 3: Pull the Alpine Docker image using Podman
      - name: Pull Docker Image with Podman
        run: |
          podman pull docker.io/library/alpine:latest

      # Step 4: Scan the Docker image using Trivy (Alpine image)
      - name: Scan Docker Image with Trivy
        run: |
          podman images  # Verify the image exists
          export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/podman/podman.sock  # Use Podman socket
          trivy image alpine:latest

    ```
## Prerequisites
Podman must be installed on the self-hosted runner. Podman is a rootless container engine, making it ideal for secure, no-sudo environments.
A self-hosted GitHub Actions runner configured with RedHat or a compatible Linux OS.
## Steps Explained
Check if Podman is Installed
The workflow first verifies if Podman is available on the runner. If Podman is not installed, the workflow will exit with an error.

Install Trivy Locally Without Sudo
Trivy is downloaded and installed locally within the user’s environment (no sudo needed). The installation path is set to $HOME/bin, and Trivy is added to the PATH for easy access in subsequent steps.

Pull Docker Image Using Podman
A sample Docker image, alpine:latest, is pulled from Docker Hub. This can be customized to any other image or registry, including private registries (e.g., Azure Container Registry, Amazon ECR).

Scan Docker Image Using Trivy
The Docker image is scanned for vulnerabilities using Trivy. The DOCKER_HOST environment variable is set to use the Podman socket, ensuring compatibility between Trivy and Podman.

Customization
You can adjust the following parameters based on your requirements:

Docker Image: Change alpine:latest in the "Pull Docker Image" step to any other image you want to scan.
Trivy Version: Modify the version of Trivy by updating the download URL in the "Install Trivy Locally" step.
Notes
This setup is suitable for environments where sudo access is restricted.
The flexibility of Podman allows this solution to work with multiple container registries (e.g., Docker Hub, Azure Container Registry).
License
This project is licensed under the MIT License.
