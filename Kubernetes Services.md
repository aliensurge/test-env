## Objective ##

Validate and simulate Uptycs frontend services such as login, UI, API, AgentQuery, and Downloads. This POC focuses on understanding the interaction between frontend pods and simulating real-world user/API behavior.

---

Components in Scope (Must-Have)

    Login

    UI

    API

    AgentQuery

    Downloads

---

### Step 1 Install Docker, kubectl & minikube ###
```bash

sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker


curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/


curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube && sudo mv minikube /usr/local/bin/


sudo yum install -y python3
pip3 install flask
```


