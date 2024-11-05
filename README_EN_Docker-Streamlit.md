# Streamlit Docker Application Setup

This document contains the necessary steps to set up an EC2 instance on AWS and deploy a Streamlit application using Docker.

## Requirements
- AWS account
- A terminal or SSH client for SSH access

## Steps

### 1. Create an AWS EC2 Instance

Log into the AWS management console and create an EC2 instance with the following specifications:

- **AMI**: Ubuntu 24.04
- **Instance Type**: t2.micro
- **Storage (Volume)**: 20 GB
- **Security Group**: 
  - TCP 8501 (for Streamlit application access)
  - TCP 22 (for SSH access)

### 2. Connect to the EC2 Instance via SSH

After creating the EC2 instance, connect to it using the following command in your terminal or SSH client:

```bash
ssh -i "your-key.pem" ubuntu@<EC2_PUBLIC_IP>
```

Ensure your `your-key.pem` file has the correct permissions (`chmod 400 your-key.pem` to set the permissions).

### 3. Install Docker

Use the following commands to install Docker:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker
docker version
```

These steps install the latest version of Docker and add the current user to the `docker` group, allowing Docker commands to run without root privileges.

### 4. Removing Docker (Optional)

If you want to remove Docker, you can use the following commands:

```bash
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
```

### 5. Create a Dockerfile

Create a Dockerfile for the image we will build.

```bash
touch Dockerfile
vim Dockerfile
```

Then, add the following content to the file:

```text
# Use Python 3.10 as the base image
FROM python:3.11.4-slim

# Set the working directory
WORKDIR /app

# Copy the requirements.txt file to install dependencies
COPY requirements.txt .

# Install the necessary packages
RUN pip install --no-cache-dir -r requirements.txt

# Copy application files to the working directory
COPY . .

# Expose the port for external access
EXPOSE 8501

# Command to start the Streamlit application
CMD ["streamlit", "run", "Car_Price_Prediction_App.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

### 6. Build the Docker Image

Build the Docker image for your application in the directory where the Dockerfile and application files are located:

```bash
docker build -t streamlit-app .
```

This command reads your `Dockerfile` and creates a Docker image named `streamlit-app`.

### 7. List Docker Images

To list the Docker images:

```bash
docker images
```

This command shows all available Docker images on the system.

### 8. Deploy the Application in a Docker Container

To run your application using the created image:

```bash
docker run -p 8501:8501 streamlit-app
```

This command starts the Streamlit application and makes it accessible on port 8501.

### 9. Run in Detached Mode

To run the application in the background (detached mode), add the `-d` flag:

```bash
docker run -d -p 8501:8501 streamlit-app
```

This command runs the application in the background, freeing up the terminal.

### 10. List Running Containers

To list all currently running Docker containers:

```bash
docker ps
```

This command shows only the running containers.

### 11. Stop a Running Container

To stop a specific container:

```bash
docker stop <container_id>
```

Replace `<container_id>` with the ID from the `docker ps` command.

### 12. List All Containers

To see all containers (including stopped ones):

```bash
docker ps -a
```

This command lists all Docker containers.

### 13. Remove a Specific Container

Before removing a container, you need to stop it. After stopping, you can remove it with:

```bash
docker rm <container_id>
docker rm $(docker ps -aq) # Removes all stopped containers.
```

Replace `<container_id>` with the ID of the container you wish to delete.

## Notes

After completing the setup steps, you can access your Streamlit application at `http://<EC2_PUBLIC_IP>:8501`.
