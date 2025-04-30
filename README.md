# FastAPI Project Setup in GitHub Codespaces

This README guides you through setting up and running the FastAPI project in GitHub Codespaces, including environment management, server operations, and Docker integration.

## 1-Running the Project in GitHub Codespaces 

### 1. Setup Codespace

- Open the GitHub repository in your browser.
- Click on the "Code" button, then select "Open with Codespaces" > "New codespace".

### 2. Environment Setup

Once your codespace is ready, set up a virtual environment and install dependencies:

```bash
# Create a virtual environment
python -m venv venv
```

```bash
# Activate the virtual environment
source venv/bin/activate
```

```bash
# Install requirements
pip install -r requirements.txt
```
### 3\. Run the Server

To run the FastAPI server on port 8000:


`uvicorn app:app --host 0.0.0.0 --port 8000 --reload`

### 4\. Port Forwarding

*   GitHub Codespaces will automatically suggest port forwarding for port 8000.
    
*   Set the port to "Public" to access the web server via the provided URL, typically formatted as:
    
    `https://port-8000-your-codespace-name.githubpreview.dev`
    

Testing the Server
------------------

Access the `/predict` endpoint using the following URL format:

`https://port-8000-your-codespace-name.githubpreview.dev/predict?text=Your%20example%20text%20here`

Replace `Your%20example%20text%20here` with the URL-encoded text you wish to analyze.

Docker Integration
------------------

### Building the Docker Image

Build the Docker image using:

```bash
docker build -t myapi .
```

### Docker Commands Cheat Sheet

#### Image Management
```bash
# Build an image from Dockerfile in current directory
docker build -t myapi .

# List all images
docker images
```

#### Container Management
```bash
# Run a container (foreground)
docker run -p 8000:80 myapi

# Run a container (background)
docker run -d -p 8000:80 myapi

# Run and auto-remove when stopped
docker run --rm -p 8000:80 myapi

# Run with a specific name
docker run -d --name myapi-container myapi

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Access container terminal
docker exec -it <container_id> sh

# Stop a container
docker stop <container_id>

# Start a stopped container
docker start <container_id>

# Remove a container
docker rm <container_id>
```

#### Flag Meanings
- `-t`: Tag an image with a name
- `-p`: Map ports (host:container)
- `-d`: Run in detached mode (background)
- `--rm`: Remove container when it stops
- `-it`: Interactive terminal
- `--name`: Assign a name to the container

### Docker Image Upload to Azure ACR

#### Prerequisites

Ensure Azure CLI is installed in your Codespace:



`pip install azure-cli`

Login to Azure:



`az login`

Login to your Azure Container Registry:



`az acr login --name <RegistryName>`

Tag and push the Docker image:



`docker tag myapi <RegistryName>.azurecr.io/myapi:v1 docker push <RegistryName>.azurecr.io/myapi:v1`

Replace `<RegistryName>` with your actual Azure Container Registry name.

### Verifying the Image in ACR

List the repositories in your ACR to verify the upload:

`az acr repository list --name <RegistryName> --output table`


# Deploying to Azure Container Instance (ACI)

## Step 1: Configure Authentication Create a service principal to securely interact with Azure services, including pulling images from ACR. This identity allows automation without using your personal credentials. 

Execute the following command to generate a service principal:  

```bash az ad sp create-for-rbac --skip-assignment```

This command outputs several key pieces of information, including `appId` and `password`. Record these securely as they will be used to authenticate and pull images.

Step 2: Grant Access to Your Image
----------------------------------

To allow the newly created service principal to access and pull images from your ACR, assign it the `AcrPull` role. Replace `<appId>` with the `appId` from the previous step:


`az role assignment create --assignee <appId> --scope myregistry.azurecr.io --role acrpull`

This step ensures that your service principal has the necessary permissions to pull the container image.

Step 3: Deploy the Container with Port Configuration
----------------------------------------------------

Deploy your container instance using the Docker image from ACR. Specify the port to expose, such as 8000, which allows external access to that port:


`az container create --resource-group myResourceGroup --name myContainer --image myregistry.azurecr.io/myimage:v1 --cpu 1 --memory 1.5 --registry-login-server myregistry.azurecr.io --registry-username <appId> --registry-password <password> --ports 8000`

Make sure to replace `<appId>` and `<password>` with the credentials obtained from creating your service principal. This command sets up the container and configures the network to allow traffic on port 8000.

Step 4: Check Your Container
----------------------------

Verify the deployment and accessibility of your container. Check its state and fetch its public IP address:

`az container show --resource-group myResourceGroup --name myContainer --query "{ProvisioningState:provisioningState,IPAddress:ipAddress}"`

This command will provide you with the status of the container instance and its publicly accessible IP address, indicating successful deployment and connectivity.


## 2-Running the Project using Azure ubuntu VM 
# 1. create a new virtual machine in Azure portal 
make sure that the VM disk capacity is more than 128GB as you will need to install alot of packages throughout the deployment phase
### 2. Environment Setup

Once your VM is ready, set up a virtual environment and install dependencies:
```bash
#install python
Sudo apt install python3-pip
sudo apt install python3.10-venv
```
```bash
# Create a virtual environment
python -m venv venv
```

```bash
# Activate the virtual environment
source venv/bin/activate
```
```bash
# clone the repo
git clone <repo URL>
```

```bash
# open the file bert-fastapi-service 
cd bert-fastapi-service
```

```bash
# Install requirements
sudo pip install -r requirements.txt
```

```bash
# install docker 
sudo apt  install docker.io
```
### 3. Docker integration
## step 1 :Building the Docker Image
```bash
# buliding the image  
sudo docker build -t myapi .
```
## step 2 : Ensure Azure CLI is installed 
```bash
# installing Azure CLI  
sudo pip install azure-cli
```
## step 3 : Login to Azure
```bash
# login to azure 
az login
```
## step 4 : create a container registery in Azure
- create a new resource which is the contaoner registery , make sure that your are using the same resource group for the VM
## step 5 : push the docker image to the registry 
```bash
# tag the image 
sudo docker tag myapi <registry name>/myapi:latest
```
```bash
# push the image 
sudo docker push <registry name>.azurecr.io/myapi
```
### Verifying the Image in ACR
```bash
# Verifying the Image in ACR 
az acr repository list --name <RegistryName> --output table
```
# Deploying to Azure Container Instance (ACI)

## Step 1: Configure Authentication Create a service principal to securely interact with Azure services, including pulling images from ACR. This identity allows automation without using your personal credentials. 

Execute the following command to generate a service principal:  

```az ad sp create-for-rbac --skip-assignment```

This command outputs several key pieces of information, including `appId` and `password`. Record these securely as they will be used to authenticate and pull images.

Step 2: Grant Access to Your Image
----------------------------------

To allow the newly created service principal to access and pull images from your ACR, assign it the `AcrPull` role. Replace `<appId>` with the `appId` from the previous step:


`az role assignment create --assignee <appId> --scope /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.ContainerRegistry/registries/<myregistry> --role acrpull
`

This step ensures that your service principal has the necessary permissions to pull the container image.

Step 3: Deploy the Container with Port Configuration
----------------------------------------------------

Deploy your container instance using the Docker image from ACR. Specify the port to expose, such as 80, which allows external access to that port:


`az container create --resource-group <ResourceGroup> --name myapi --image <regname>.azurecr.io/myapi:latest --cpu 1 --memory 1.5 --registry-login-server <myregistry>.azurecr.io --registry-username <regname> --registry-password <password> --ports 80`

Make sure to replace `<regname>` and `<password>` with the credentials obtained from creating your service principal. This command sets up the container and configures the network to allow traffic on port 80.

Step 3: Testing
----------------------------------------------------
you will get an IP address from the last step output use this IP address with port 80 `IP adress:80` and you should get the welcome message for the app 
