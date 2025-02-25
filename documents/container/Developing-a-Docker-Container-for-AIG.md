# Developing a Docker Container for AIG

Document Version: V1.0

##### Change Log

| Version | Date       | Content          |
| ------- | ---------- | ---------------- |
| 1.0     | 2025-01-10 | Document created |

##### Applicable Products
| Product | Version |
| ------- | ------- |
| AIG-302 | 1.0 |

##### Supported Environments

* Linux-based systems, ARM32 architecture

##### Prerequisites

* Access to the system via the Linux command line with sudo privileges


## Purpose

AIG is a versatile programmable product that offers three distinct programming methods to accommodate various user needs:

1. No-Code</br>
    Use the Logic Engine to perform calculations with simple formulas on a virtual tag or a current writable tag, enabling basic logic processing without writing any code.

2. Low-Code</br>
    Through the Function feature, utilize the system SDK with Python to perform lightweight programming, suitable for simple custom function development.

3. Custom Traditional Development</br>
    Supports native programming or the import of Docker images, enabling deeper and more flexible custom development.

This guide focus on custom traditional development to create a docker image for AIG.

## Environment Setup and Requirements
To ensure a smooth Docker development process for the AIG platform, set up an appropriate development environment with the necessary tools and packages as outlined below.

### 1. Development Operating System
To develop Docker images optimized for AIG, you’ll need a Linux-based operating system equipped with a compatible Docker engine. Choose from one of the following Docker engines based on your development setup:

* Docker CE (Community Edition)</br>
    Ideal for most development environments, offering a widely supported and feature-rich platform for Docker image creation.

* Moby</br>
    Moby is a lightweight, open-source container platform that aligns with the AIG product’s pre-installed environment.

* Other Compatible Docker Engines</br>
    Ensure that your chosen Docker engine is compatible with ARM architecture and can support multi-architecture builds.

Install one of these Docker engines on your Linux OS to enable Docker commands and container management in your environment.

### 2. ARM Architecture Support
Since AIG-302 runs on an ARM-based architecture, you will need to install additional tools to build and test ARM-compatible Docker images on a standard development machine. Use the following setup to emulate ARM environments during development:

* QEMU for ARM Support</br>
    Install qemu-user-static on Debian-based systems to enable emulation of ARM architecture on your machine. This will allow you to build ARM images and ensure compatibility with the AIG-302 platform.
    ```bash
    sudo apt-get update
    sudo apt-get install -y qemu-user-static
    ```

### 3. Enable Multi-Architecture Support in Docker
If you are developing on an x86_64 system and need to build or test ARM images, enable QEMU multi-architecture support to allow Docker to run ARM-based containers. This setup is essential for cross-platform development:
```bash
sudo docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
```
This command installs QEMU emulation support for various architectures, making it possible to build and run ARM containers seamlessly on a non-ARM host.

For additional details and options, please refer to the official Docker documentation on Multi-Platform Builds.

## Developing REST API Communication
In this section, we’ll set up a Docker container for AIG that performs basic REST API communication. The container will include a Python script that sends a sample REST request to AIG.

For detailed information on available API endpoints and parameters, please refer to the AIG REST API Documentation.

* [AIG-302 REST API](https://tpe-tiger.github.io/AIG302/V1.0.0/#tag/azure-iotedge/paths/~1azure-iotedge~1services/post)

### 1. Create a Simple REST Client in Python
First, create a Python script (app.py) to handle REST API communication:
```python
import requests
 
# Set the base URL for the AIG REST API
API_BASE_URL = "http://172.31.0.1:59000/api/v1"
API_TOKEN = "YOUR_API_TOKEN"
 
# Example function to retrieve a tag value from AIG
def get_tag_value(provider, source, tag_name):
    url = f"{API_BASE_URL}/tags/monitor/{provider}/{source}?tags={tag_name}"
    headers = {
        "Authorization": f"Bearer {API_TOKEN}"
    }
    try:
        response = requests.get(url, headers=headers, verify=False)  # verify=False for self-signed certs
        response.raise_for_status()
        print(f"Tag Value: {response.json()}")
    except requests.RequestException as e:
        print(f"Error fetching tag value: {e}")
 
if __name__ == "__main__":
    # Replace 'system' with required provider, 'status' with target source and 'cpuUsage' with the actual tag name
    get_tag_value("system", "status", "cpuUsage")
```

This script connects to the AIG API using an API token, retrieves a tag value, and prints it to the console. Please replace "YOUR_API_TOKEN" with the token generated as document [Generating an API token for application development](../Generating-an-API-Token-for-Application-Development.md).

> Note: For secure API communication within the container, use http://172.31.0.1:59000 as the base URL.

### 2. Write a Dockerfile for ARM32 Architecture
Create a Dockerfile to package this application. Since AIG-302 is ARM32, we’ll use a Python image for the ARM32 architecture.
```Dockerfile
# Use ARM32v7 Python base image
FROM arm32v7/python:3.8-slim
 
# Install necessary dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*
 
# Set the working directory
WORKDIR /usr/src/app
 
# Copy application files to the container
COPY app.py ./
 
# Install required Python packages
RUN pip install requests
 
# Set the command to run the Python script
CMD ["python", "./app.py"]
```

This Dockerfile performs the following steps:
* Uses an ARM32v7 Python image to match AIG-302’s architecture.
* Installs necessary dependencies (curl and requests library).
* Copies the Python script (app.py) into the container.
* Sets the container command to run the script.

### 3. Build the Docker Image for ARM32 Architecture
To build the Docker image on a development system (e.g., x86_64), enable multi-architecture builds.

1. Enable multi-architecture support:
    ```bash
    sudo docker run --privileged --rm tonistiigi/binfmt --install all
    ```

2. Build the image for ARM32 using buildx:
    ```bash
    sudo docker buildx build --platform linux/arm/v7 -t my-arm32-rest-client:latest .
    ```
    This command creates an ARM32-compatible Docker image named my-arm32-rest-client.

3. Check if the image created:
    ```bash
    sudo docker images my-arm32-rest-client:latest
    ```

### 4. Deploying and Running the Container on AIG
Once the image is built, transfer it to the AIG device. There are several methods for transferring Docker images to another system, such as AIG. You can either push the image to a Docker registry or save and load it manually.

For official documentation on Docker image transfer methods, please refer to the Docker Image Management Documentation.

To run the container on AIG:
```bash
sudo docker run --rm --name rest-client my-arm32-rest-client:latest
```

This container will execute the app.py script, which communicates with the AIG REST API.

> Note: For setting up API tokens, please refer to document "Create an API token for application development". Additionally, for REST API calls within the container, always use http://172.31.0.1:59000 as the AIG host URL.

### 5. Deploying in a Production Environment
To deploy the application in a production environment, you can use Docker Compose to manage the container with appropriate configurations for persistent storage, port mapping, and restart policies.

1. Before using Docker Compose, ensure it is installed on your system. For Debian-based systems, use the following commands:
    ```bash
    sudo apt-get update
    sudo apt-get install -y docker-compose
    ```
    Verify the installation by checking the version:
    ```bash
    sudo docker-compose --version
    ```
2. Create a docker-compose.yml file with the necessary configurations. Below is an example tailored for the A2 environment:
    ```yaml
    version: "3.3"
    
    services:
      rest-client:
        image: my-arm32-rest-client:latest
        container_name: rest-client
        restart: always
        ports:
          - "12345:12345"  # Map host port 12345 to container port 12345
        volumes:
          - /mnt/sdcard/data:/app/data   # Mount SD card for persistent storage
          - ./config:/app/config         # Bind mount for configuration files
        environment:
          - API_TOKEN=your-api-token     # Pass API token as an environment variable
        command: ["python", "./app.py"]  # Command to execute in the container (Will override CMD defined in Dockerfile)
    ```
   * restart: always: Ensures the container automatically restarts if it crashes or if the system reboots.
   * volumes:
     - /mnt/sdcard/data:/app/data: Mounts the SD card directory (/mnt/sdcard/data) to /app/data in the container for persistent data storage.
     - ./config:/app/config: Mounts a local config directory to /app/config in the container for configuration files.
   * ports: Maps port 12345 on the host to port 12345 in the container, allowing external access to the application.
   * environment: Passes sensitive information (e.g., API tokens, base URLs) to the container in a secure manner.
3. To deploy the application, use the docker-compose command:
    ```bash
    sudo docker-compose up -d
    ```
    This command:
    * Starts the container in detached mode (-d) so it runs in the background.
    * Automatically applies the restart policy defined in the docker-compose.yml file.
4. Verify the Container Status</br>
    Ensure the container is running correctly:
    ```bash
    sudo docker-compose ps
    ```
    Check the logs if needed:
    ```bash
    sudo docker-compose logs -f
    ```

#### References
* [Docker Compose - Quickstart](https://docs.docker.com/compose/gettingstarted/#step-2-define-services-in-a-compose-file)
* [Docker Compose - Use Compose in production](https://docs.docker.com/compose/how-tos/production/)
* [Docker Compose - restart parameter](https://docs.docker.com/reference/compose-file/services/#restart)