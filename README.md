📁 Wiki Page: Implementation-Guide.md
markdown
Copy
Edit
# 📦 Implementation Guide

This section walks through the complete deployment and configuration of OpenVPN on an AWS EC2 instance running Ubuntu Server 22.04 LTS.

---

## 1. Launch EC2 Instance

- **Operating System**: Ubuntu 22.04 LTS
- **Instance Type**: t2.micro (Free Tier eligible)
- **Ports to Open (Security Group)**:
  - **22** (TCP) for SSH
  - **1194** (UDP) for OpenVPN

---

## 2. Install OpenVPN and Easy-RSA

```bash
sudo apt update
sudo apt install openvpn easy-rsa -y
3. Generate Keys and Certificates
bash
Copy
Edit
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
source vars
./clean-all
./build-ca          # Create Certificate Authority
./build-key-server server
./build-dh
openvpn --genkey --secret keys/ta.key
./build-key client1 # For client
Replace client1 with a meaningful name for each client.

4. Configure OpenVPN Server
Edit /etc/openvpn/server.conf with the following:

conf
Copy
Edit
port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh.pem
tls-auth ta.key 0
cipher AES-256-CBC
persist-key
persist-tun
user nobody
group nogroup
status openvpn-status.log
log /var/log/openvpn.log
verb 3
5. Enable IP Forwarding & NAT
Enable IP forwarding:

bash
Copy
Edit
echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
Configure firewall rules:

bash
Copy
Edit
sudo ufw allow 1194/udp
sudo ufw allow OpenSSH
sudo ufw enable
Set up NAT in /etc/ufw/before.rules:

bash
Copy
Edit
*nat
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
COMMIT
6. Start and Enable the VPN Server
bash
Copy
Edit
sudo systemctl start openvpn@server
sudo systemctl enable openvpn@server
Check if the VPN is active:

bash
Copy
Edit
sudo systemctl status openvpn@server
7. Generate Client Profiles
Use the following script to generate a unified .ovpn file for clients:

bash
Copy
Edit
cd ~/openvpn-ca/keys

cat > client1.ovpn <<EOF
client
dev tun
proto udp
remote YOUR_EC2_PUBLIC_IP 1194
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-CBC
verb 3
<ca>
$(cat ca.crt)
</ca>
<cert>
$(cat client1.crt)
</cert>
<key>
$(cat client1.key)
</key>
<tls-auth>
$(cat ta.key)
</tls-auth>
key-direction 1
EOF
Replace YOUR_EC2_PUBLIC_IP with your EC2 instance's Elastic IP.

yaml
Copy
Edit

---

## 📁 Wiki Page: `Testing-Results.md`

```markdown
# 🧪 Testing & Results

This section describes how the system was tested and the results of performance and security checks.

---

## ✅ Testing Steps

- ✔️ Successfully connected from external client using OpenVPN GUI.
- ✔️ Pinged EC2 internal and private IPs.
- ✔️ DNS and routing tested over VPN tunnel.
- ✔️ Wireshark confirmed encrypted packet transmission.

---

## 📊 Performance Metrics

| Metric           | Result       |
|------------------|--------------|
| Latency          | ~30–40ms     |
| VPN Uptime       | 99.9%        |
| Connected Clients| 3            |
| Packet Encryption| Verified     |

---

## 🔒 Security Checks

- ✅ **Nmap Scan**: Only SSH and VPN ports open
- ✅ **Encrypted Tunnel**: Verified with packet inspection tools
- ✅ **Fail2Ban**: Configured to block brute-force SSH attempts

---

# 🚀 Future Improvements

- ✅ Deploy **OpenVPN Access Server** for web-based management.
- ✅ Automate provisioning using **Terraform** or **CloudFormation**.
- ✅ Integrate with **LDAP/Active Directory** for centralized authentication.
- ✅ Enable **Two-Factor Authentication (2FA)**.
- ✅ Implement system monitoring with **Prometheus & Grafana**.
- ✅ Schedule automatic **certificate/key rotation**.

---

# 📚 References

- 📘 [OpenVPN Documentation](https://openvpn.net/community-resources/)
- 📘 [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- 📘 [Easy-RSA GitHub](https://github.com/OpenVPN/easy-rsa)
- 📘 [Ubuntu Server Docs](https://ubuntu.com/server/docs)

---

# 👨‍💻 About the Author

**Mohammad Qasim Matloob**  
BSc (Hons) Cyber Security  
**Buckinghamshire New University**

- 💼 GitHub: [MohammadQasim03](https://github.com/MohammadQasim03)  
- 🔗 LinkedIn: [linkedin.com/in/mohammad-matloob-b3a880253](https://www.linkedin.com/in/mohammad-matloob-b3a880253)  
- ✉️ Email: Qasimmatloob786@gmail.com  

> _This project was completed as part of a final year dissertation on secure remote access using cloud infrastructure._

---

![Project Screenshot](https://github.com/user-attachments/assets/1d42649b-6050-4a7e-878f-97bd718ab03b)
