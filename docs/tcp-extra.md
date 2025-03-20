# **OpenVPN Setup with Docker**  

This guide provides step-by-step instructions to set up an OpenVPN server using Docker with a **TCP connection**.  

---

## **Prerequisites**  
Ensure you have **Docker** installed on your system.  

### **1️⃣ Create a Docker Volume for OpenVPN Data**  
```sh
docker volume create --name ovpn-data
```
This volume stores OpenVPN configuration files and certificates persistently.  

---

### **2️⃣ Generate OpenVPN Configuration**  
```sh
docker run --rm \
  -v ovpn-data:/etc/openvpn \
  pankajthakur888/openvpn \
  ovpn_genconfig -u tcp://$(hostname -I | awk '{print $1}'):443
```
- Uses **TCP** on port `443` (you can change it if needed).  
- Dynamically retrieves the **private IP address** for the VPN server.  

> 📝 If you need to use the **public IP**, replace `$(hostname -I | awk '{print $1}')` with `$(curl -s ifconfig.me)`.  

---

### **3️⃣ Initialize Public Key Infrastructure (PKI)**  
```sh
docker run --rm -it \
  -v ovpn-data:/etc/openvpn \
  pankajthakur888/openvpn \
  ovpn_initpki
```
This step sets up the **certificate authority (CA)** and generates necessary keys.  

---

### **4️⃣ Start the OpenVPN Server**  
```sh
docker run -d \
  -v ovpn-data:/etc/openvpn \
  -p 443:1194/tcp \
  --cap-add=NET_ADMIN \
  pankajthakur888/openvpn
```
- Runs the server in **detached mode (`-d`)**.  
- Maps **port 443** on the host to **1194/tcp** inside the container.  
- Grants **network administration privileges** (`--cap-add=NET_ADMIN`).  

---

### **5️⃣ Create a Client Certificate**  
```sh
docker run --rm -it \
  -v ovpn-data:/etc/openvpn \
  pankajthakur888/openvpn \
  easyrsa build-client-full pankaj nopass
```
- Generates a **client certificate** named `pankaj`.  
- `nopass` means the client certificate **won’t require a password** when connecting.  

---

### **6️⃣ Generate Client Configuration File (`.ovpn`)**  
```sh
docker run --rm \
  -v ovpn-data:/etc/openvpn \
  pankajthakur888/openvpn \
  ovpn_getclient pankaj > pankaj.ovpn
```
- Saves the client configuration to `pankaj.ovpn`, which can be used to connect to the VPN.  

---

## **🔗 Connecting to the VPN**  
1. Download the **`pankaj.ovpn`** file to your **client device**.  
2. Import it into your **OpenVPN client** (Windows, macOS, Linux, or mobile apps).  
3. Connect and enjoy a **secure VPN connection**!  

---

## **🎯 Notes**
- **Modify the port** if needed (`443` → custom port).  
- **To remove OpenVPN data**, run:  
  ```sh
  docker volume rm ovpn-data
  ```
- **Check logs** if the server fails to start:  
  ```sh
  docker logs $(docker ps -q --filter "ancestor=pankajthakur888/openvpn")
  ```

🚀 **Enjoy your secure VPN!** 🚀  

---
