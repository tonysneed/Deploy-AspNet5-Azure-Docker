# Visual Studio Code Setup
## For ASP.NET 5 on Azure with Docker

1. Download and install **Visual Studio Code** on Windows

  - Instructions: https://code.visualstudio.com/Docs/setup
  - Download **Visual Studio Code for Windows**
  - Launch `VSCodeSetup.exe`
    + The install location is `C:\Users\username\AppData\Local\Code`.
    + From the console you can type `code .` to open VSCode on that folder.
    + You might need to log off and on again for the change in `PATH` to take effect.
    
2. Install **Docker Client for Windows**

  - Install Docker Client for Windows using Chocolatey
    + This installs just the Docker client, not the Docker engine for hosting containers
  - Open an admin command prompt, then enter:
    ```
    choco install docker -y
    ```
  - To verify Docker installation enter: `docker`
    + You should see a list of docker commands
    
3. Create a Linux Virtual Machine on the Azure Portal
  - Go to the Azure portal: `https://portal.azure.com`
  - If not a subscriber sign up for a free trial.
  - Create a new VM using the image: `Ubuntu Server 14.04 LTS`
    + Accept the default username: `azureuser`
    + Enter a password (or upload an SSH key)
    + Note the DNS name, for example: `docker-aspnet5.cloudapp.net`

4. Create a CA, server and client keys with OpenSSL
  - Instructions: http://docs.docker.com/articles/https
  - Install OpenSSL for Windows: http://slproweb.com/products/Win32OpenSSL.html
    + First install `Visual C++ 2008 Redistributables (x64)`
    + Then install `Win64 OpenSSL v1.0.2a` 
  - Generate CA private and public keys (specify `docker-aspnet5.cloudapp.net` for host name or CN):
    + Create a folder to contain the certificates
    + Use any password when prompted
    + Press Enter when prompted for certificate info,
      except for Common Name enter: `docker-aspnet5.cloudapp.net`
    ```
    openssl genrsa -aes256 -out ca-key.pem 2048
    openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
    ```
  - Create server key and certificate signing request
    ```
    openssl genrsa -out server-key.pem 2048
    openssl req -subj "/CN=docker-aspnet5.cloudapp.net" -new -key server-key.pem -out server.csr
    ```
  - Sign the public key with the CA cert
    ```
    echo subjectAltName = IP:10.10.10.20,IP:127.0.0.1 > extfile.cnf
    openssl x509 -req -days 365 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf
    ```
  - Create a client key and certificate signing request
    ```
    openssl genrsa -out key.pem 2048
    openssl req -subj '/CN=client' -new -key key.pem -out client.csr
    ```
  - Create an extensions config file
    ```
    echo extendedKeyUsage = clientAuth > extfile.cnf
    ```
  - Sign the public key
    ```
    openssl x509 -req -days 365 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile.cnf
    ```
  - Remove the two certificate signing requests
    ```
    rm -v client.csr server.csr
    ```
  - Make keys read-only
    ```
    chmod -v 0400 ca-key.pem key.pem server-key.pem
    ```
  - Make certificates read-only
    ```
    chmod -v 0444 ca.pem server-cert.pem cert.pem
    ```
  - Encode CA cert, server cert and server key files
    + Download `base64.zip`: http://www.fourmilab.ch/webtools/base64
    + Extract `base64.exe` to folder containing cert and key files
    ```
    base64 ca.pem > ca64.pem
    base64 server-cert.pem > server-cert64.pem
    base64 server-key.pem > server-key64.pem
    ```

5. Add the Docker VM Extension
  - Instructions: http://azure.microsoft.com/en-us/documentation/articles/virtual-machines-docker-with-portal
  - Add the Docker VM extension
    + Upload base64 encoded cert and key files created earlier
  - Add the Docker Communication Endpoint
    + Enter a name for the endpoint (for example, docker)
    + Enter 4243 for both private and public ports.
    + Leave the protocol value showing TCP
    + Click OK to create the endpoint
    + Wait until the docker extension and endpoint have both been added
  - Verify docker installation. Using docker client enter:
    ```
    docker --tls -H tcp://docker-aspnet5.cloudapp.net:4243 info
    ```

6. Set docker host environment variable
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
    
7. Run **console sample**
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
  
8. Run **web sample**
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

