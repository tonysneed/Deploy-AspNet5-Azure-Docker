## How to Create Keys and Certs with OpenSSL

NOTE: This procedure isn't necessary if you select a VM from the Azure marketplace which *already has Docker installed*.

- Instructions: http://docs.docker.com/articles/https

1. Install OpenSSL for Windows: http://slproweb.com/products/Win32OpenSSL.html
  + First install `Visual C++ 2008 Redistributables (x64)`
  + Then install `Win64 OpenSSL v1.0.2a`
  
2. Generate CA private and public keys (specify `docker-aspnet5.cloudapp.net` for host name or CN):
  + Create a folder to contain the certificates
  + Use any password when prompted
  + Press Enter when prompted for certificate info,
    except for Common Name enter: `docker-aspnet5.cloudapp.net`
    
    ```
    openssl genrsa -aes256 -out ca-key.pem 2048
    openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
    ```
    
3. Create server key and certificate signing request

    ```
    openssl genrsa -out server-key.pem 2048
    openssl req -subj "/CN=docker-aspnet5.cloudapp.net" -new -key server-key.pem -out server.csr
    ```
    
4. Sign the public key with the CA cert

    ```
    echo subjectAltName = IP:10.10.10.20,IP:127.0.0.1 > extfile.cnf
    openssl x509 -req -days 365 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf
    ```
    
5. Create a client key and certificate signing request

    ```
    openssl genrsa -out key.pem 2048
    openssl req -subj '/CN=client' -new -key key.pem -out client.csr
    ```
    
6. Create an extensions config file

    ```
    echo extendedKeyUsage = clientAuth > extfile.cnf
    ```

7. Sign the public key

    ```
    openssl x509 -req -days 365 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile.cnf
    ```
    
8. Remove the two certificate signing requests

    ```
    rm -v client.csr server.csr
    ```
    
9. Make keys read-only

    ```
    chmod -v 0400 ca-key.pem key.pem server-key.pem
    ```
    
10. Make certificates read-only

    ```
    chmod -v 0444 ca.pem server-cert.pem cert.pem
    ```
    
11. Encode CA cert, server cert and server key files

  + Download `base64.zip`: http://www.fourmilab.ch/webtools/base64
  + Extract `base64.exe` to folder containing cert and key files
  
    ```
    base64 ca.pem > ca64.pem
    base64 server-cert.pem > server-cert64.pem
    base64 server-key.pem > server-key64.pem
    ```

