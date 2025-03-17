# Create-a-Multibranch-Pipeline
1. Create three branches (main, stag, dev):
git switch -c dev //Create and Switch to a New Branch
git push -u origin dev //push the new branch
git switch -c stag
git push -u origin stag
git switch main   //return to the main branch

2. Create 3 Namespaces:
kubectl create namespace main
kubectl create namespace stag
kubectl create namespace dev

3. Create ServiceAccounts for Each Namespace
user used by Jenkins to interact with the Kubernetes API where Jenkins will deploy resources

kubectl create serviceaccount jenkins-deployer -n main
kubectl create serviceaccount jenkins-deployer -n stag
kubectl create serviceaccount jenkins-deployer -n dev

4. Assign Roles to the ServiceAccounts
Create a Role and RoleBinding for each namespace.

kubectl apply -f role-main.yaml
kubectl apply -f rolebinding-main.yaml

kubectl apply -f role-stag.yaml
kubectl apply -f rolebinding-stag.yaml

kubectl apply -f role-dev.yaml
kubectl apply -f rolebinding-dev.yaml

5. Get the Token for Each ServiceAccount:
For each namespace, create a Secret and link it to the ServiceAccount.
#Apply the Secret file :
kubectl apply -f jenkins-deployer-secret-main.yaml

#Link the Secret to the ServiceAccount:
kubectl -n main patch serviceaccount jenkins-deployer -p '{"secrets": [{"name": "jenkins-deployer-token-main"}]}'

#Extract Tokens for Each Namespace
kubectl -n main get secret jenkins-deployer-token-main -o jsonpath='{.data.token}' | base64 --decode

and the same thing for other namespaces

6. Add Tokens as Credentials in Jenkins
Jenkins needs these tokens to access Kubernetes. 
Go to Jenkins → Manage Jenkins → Manage Credentials.
Click Add Credentials.
Choose Secret text.
Paste the token for the main namespace and give it an ID like main-namespace-token.
Repeat for stag and dev namespaces, using IDs like stag-namespace-token and dev-namespace-token.
7. Create a Jenkins Pipeline:
Checkout Code: Pulls the code from GitHub repository.

Build Docker Image: Builds a Docker image your app.

Push Docker Image: Pushes the image to Docker Hub.

Deploy to Kubernetes: Deploys the app to the correct namespace (main, stag, or dev) based on the branch.

8. Use a Shared Library:

9. the agent is installed in the previous task:
     The agent must have kubectl installed.
     The agent must have the kubeconfig file (which contains the cluster details and credentials).
Copy the Kubeconfig File to the Agent:
On the agent:
```bash
mkdir -p ~/.kube 
chmod 700 ~/.kube
```
On the master:
```bash
chmod 600 ~/.kube/config
scp ~/.kube/config jenkins@192.168.105.11:~/.kube/config
scp -r ~/.minikube jenkins@192.168.105.11:~/

```
 Verify Access:
 on the agent:
 kubectl get pods --namespace=main
