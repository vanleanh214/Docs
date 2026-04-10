## =======Cai dat ban phoi k8s (rke2)===========
**1.In Master-k8s-01**

Step 1: Install repository
```bash
curl -sfL https://get.rke2.io | INSTALL_RKE2_METHOD=tar sh -
```
Step 2: Create & Modirfile file config
```bash
mkdir -p /etc/rancher/rke2 && vi /etc/rancher/rke2/config.yaml
```
Step 3: Modirfile config.yaml
```bash
tls-san:
  - k8s.cluster.local # IP VIP Load Balancer
  - 103.171.92.247
write-kubeconfig-mode: "0644"
```
Step 4: Start service
```bash
systemctl start rke2-server
```
**2.In Master-k8s-02**

Step 1: Install repository
```bash
curl -sfL https://get.rke2.io | INSTALL_RKE2_METHOD=tar sh -
```
Step 2: Create & Modirfile file config
```bash
mkdir -p /etc/rancher/rke2 && vi /etc/rancher/rke2/config.yaml
```
Step 3: Get token of master-k8s-01
```bash
cat /var/lib/rancher/rke2/server/node-token
```
Step 4: Modifile file config.yaml
```bash
server: https://IP-VIP:<PORT-SUPERVISOR>
token: <CHUOI_TOKEN_VỪA_COPY>
tls-san:
  - IP-VIP
```
Step 5: Start service
```bash
systemctl start rke2-server
```
Noted: Check logs init
```bash
journalctl -u rke2-server -f
```
**3.In Worker**

Step 1: Install repository
```bash
curl -sfL https://get.rke2.io | INSTALL_RKE2_METHOD=tar sh -
```
Step 2: Create & Modirfile file config
```bash
mkdir -p /etc/rancher/rke2 && vi /etc/rancher/rke2/config.yaml
```
Step 3: Get token of master-k8s-01
```bash
cat /var/lib/rancher/rke2/server/node-token
```
Step 4: Modifile file config.yaml
```bash
server: https://IP-VIP:<PORT-SUPERVISOR>
token: <CHUOI_TOKEN_VỪA_COPY>
```
Step 5: Start service
```bash
systemctl enable --now rke2-agent
```
**4.Export to using kubectl** 
```bash
echo 'export PATH=$PATH:/var/lib/rancher/rke2/bin' >> ~/.bashrc

echo 'export KUBECONFIG=/etc/rancher/rke2/rke2.yaml' >> ~/.bashrc

source ~/.bashrc
```
**5.Remove worker node**

Step 1:
```bash
kubectl drain <TÊN_NODE_WORKER> --ignore-daemonsets --delete-emptydir-data
```
Step 2:
```bash
kubectl delete node <TÊN_NODE_WORKER>
``` 
