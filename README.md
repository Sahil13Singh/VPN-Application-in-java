# VPN Project

## Overview
This VPN (Virtual Private Network) software is built using Java and OpenSSL. It facilitates secure communication between a client and a server by encrypting data and forwarding it to the target host.

## How to Run the VPN

### Prerequisites
- Java Development Kit (JDK) 8 or later
- OpenSSL

### Running the Server
To run the server, execute the `ForwardServer` class with the following program arguments:

```bash
--handshakeport=2206 --usercert=certs/server.pem --cacert=certs/CA.pem --key=certs/serverprivatekey.pem
```

### Running the Client
To run the client, execute the client class with the following program arguments:

```bash
--handshakehost=localhost --handshakeport=2206 --targethost=localhost --targetport=1337 --usercert=certs/client.pem --cacert=certs/CA.pem --key=certs/clientprivatekey.der
```

## How to Test the VPN

1. Open a netcat listener on the target port specified by the client:
   ```bash
   nc -l 1337
   ```

2. After the handshake is complete, check the client logs for the forward port and connect a netcat to this port:
   ```bash
   nc localhost <forward-port>
   ```

3. You should now be able to exchange text between the two netcats, demonstrating that the VPN is forwarding the data.

4. To test the encryption, uncomment the specified block of code in the `ForwardThread` file.

## Certificates

This VPN software requires that both the server's and client's certificates are signed by the same Certificate Authority (CA).

### Generate CA Certificate and Private Key
```bash
openssl req -new -x509 -newkey rsa:2048 -keyout CAprivatekey.pem -out CA.pem
```

### Generate Client/Server CSR
```bash
openssl req -out client.csr -new -newkey rsa:2048 -keyout clientprivatekey.pem
```

### Sign the CSR with the CA
```bash
openssl x509 -req -in client.csr -CA CA.pem -CAkey CAprivatekey.pem -CAcreateserial -out client.pem
```

### Convert .pem Key to .der
```bash
openssl pkcs8 -nocrypt -topk8 -inform PEM -in clientprivatekey.pem -outform DER -out clientprivatekey.der
```

## How Does It Work?

1. **ClientHello**: The client sends the initial message to the server containing:
   - `MessageType = ClientHello`
   - `Certificate = client’s x509 certificate (as a string)`

2. **Server Verifies Client Certificate**.

3. **ServerHello**: If the client certificate is verified, the server responds with:
   - `MessageType = ServerHello`
   - `Certificate = server’s x509 certificate`

4. **Client Verifies Server Certificate**.

5. **Client Requests Port Forwarding** to target host and port:
   - `MessageType = forward`
   - `TargetHost = server.kth.se` (target server)
   - `TargetPort = 6789` (port of the target server)

6. **Server Sets Up Session** if it agrees on the target:
   - Generate Session Key & IV
   - Encrypt Session Key & IV with client's public key
   - Create socket endpoint and port number for session communication

7. **Server Sends Session Message**:
   - `MessageType = session`
   - `SessionKey = AES key encrypted with client’s Public Key`
   - `SessionIV = AES CTR IV, encrypted with client’s Public Key`
   - `ServerHost = address of VPN server`
   - `ServerPort = VPN Server’s port`

8. **Client Receives Session Message** and completes the handshake.

9. **VPN Client Connects with User**:
   - VPN client decides on internal communication host and port for the user
   - User creates TCP connection with client host and client port

10. **Data Transmission**:
    - User data is sent through the VPN client, encrypted, sent to the VPN server, decrypted, and finally forwarded to the target.

## Contributor
- Sahil Singh


---
