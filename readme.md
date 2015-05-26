# Deploy ASP.NET 5 Apps
## To a Linux VM with Docker on Azure

1. Install **Docker Client for Windows**

  - Install Docker Client for Windows using Chocolatey
    + This installs just the Docker client, not the Docker engine for hosting containers
  - Open an admin command prompt, then enter:
    ```
    choco install docker -y
    ```
  - To verify Docker installation enter: `docker`
    + You should see a list of docker commands
    
2. Create a Linux Virtual Machine on the Azure Portal
  - Go to the Azure portal: `https://portal.azure.com`
  - If not a subscriber sign up for a free trial.
  - Create a new VM using the image: `Ubuntu Server 14.04 LTS`
    + Accept the default username: `azureuser`
    + Enter a password (or upload an SSH key)
    + Note the DNS name, for example: `docker-aspnet5.cloudapp.net`
  - Verify docker installation. Using docker client enter:

    ```
    docker --tls -H tcp://docker-aspnet5.cloudapp.net:4243 info
    ```

3. Set docker host environment variable
  - Enter `docker info`
    + You'll see a message:
    ```
    An address incompatible with the requested protocol was used..
    Are you trying to connect to a TLS-enabled daemon without TLS?
    ```
  - Enter the following:
    ```
    set docker_host=tcp://docker-aspnet5.cloudapp.net:4232
    ```
  - This time specify TLS when requesting docker info: `docker --tls info`
    
4. Run **console sample**
  - Create ConsoleApp folder
  - Go to the aspnet repo: https://github.com/aspnet/Home/tree/dev/samples/latest/ConsoleApp
  - Download: project.json, program.cs
  - Add a file called `Dockerfile` in the app directory with the following contents:
    ```
    FROM microsoft/aspnet:1.0.0-beta4
    COPY . /app
    WORKDIR /app
    RUN ["dnu", "restore"]
    
    ENTRYPOINT ["dnx", ".", "run"]
    ```
  - From the app directory, build the docker image
    ```
    docker build -t consoleapp .
    ```
  - To verify enter: `docker images`
  - Run the container:

    ```
    docker run -t consoleapp
    ```
  - You should see the followint output: `Hello World`
  
5. Run **web sample**
  - Create WebApp folder
  - Go to the aspnet repo: https://github.com/aspnet/Home/tree/dev/samples/latest/HelloWeb
  - Download: project.json, startup.cs
  - Add a file called `Dockerfile` in the app directory with the following contents:

    ```
    FROM microsoft/aspnet:1.0.0-beta4
    COPY . /app
    WORKDIR /app
    RUN ["dnu", "restore"]
    
    EXPOSE 5004
    ENTRYPOINT ["dnx", ".", "kestrel"]
    ```
  - From the app directory, build the docker image

    ```
    docker build -t webapp .
    ```
  - To verify enter: `docker images`
  - Run the container:

    ```
    docker run -t -d -p 5004:5004 webapp
    ```
  - To verify enter: `docker ps`
  - Open a browser and go to: `http://localhost:5004`

