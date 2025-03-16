# **Prolog Web Server on AWS**

This document explains how to deploy a **web server with SWI-Prolog on AWS**, ensuring the service remains stable and accessible via **port 80**.

---

## **1️⃣ Setting Up the AWS Instance**

### **1.1 Accessing the EC2 Instance**
1. Log in to **AWS Console** → Go to **EC2** → Instances.
2. Connect via **SSH**:
   ```bash
   ssh -i "your-key.pem" ubuntu@your-public-ip
   ```

### **1.2 Installing SWI-Prolog**
```bash
sudo apt update
sudo apt install swi-prolog -y
```
Verify the installation with:
```bash
swipl --version
```

---

## **2️⃣ Creating the Prolog Web Server**

### **2.1 Writing the `server.pl` File**
```prolog
:- use_module(library(http/thread_httpd)).
:- use_module(library(http/http_dispatch)).
:- use_module(library(http/http_json)).

server(8080) :- http_server(http_dispatch, [port(8080)]).

:- http_handler(root(query), respond, []).

respond(_Request) :-
    reply_json_dict(_{message: "Hello from Prolog on AWS"}).

:- initialization(server(8080)).
```
Save the file as `/home/ubuntu/server.pl`.

### **2.2 Running the Server**
```bash
swipl -q -f /home/ubuntu/server.pl
```
Test in the terminal:
```bash
curl http://localhost:8080/query
```
If you see `{"message":"Hello from Prolog on AWS"}`, the server is working.

---

## **3️⃣ Making the Server Public**

### **3.1 Opening Port 8080 on AWS**
1. Go to **Security Groups** in AWS.
2. Add an **Inbound Rule**:
   - **Type:** HTTP
   - **Protocol:** TCP
   - **Port:** `8080`
   - **Source:** `0.0.0.0/0`

### **3.2 Redirecting Port 80 to 8080 with `iptables`**
Run:
```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
```
Make it **persistent** across reboots:
```bash
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
```
Now, `http://your-public-ip/query` automatically redirects to **Prolog on port 8080**.

---

## **4️⃣ Keeping the Server Always Running**

### **4.1 Creating a `systemd` Service**

```bash
sudo nano /etc/systemd/system/prolog.service
```

📌 **Contents:**
```ini
[Unit]
