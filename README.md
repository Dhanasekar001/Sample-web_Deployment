# Sample-web_Deployment
## Step 1: Setup your Project and create a HTML file
1. Create a Project folder
2. create a index.html file:
```
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple UI App</title>
</head>
<body>
    <h1>Hello, World!</h1>
    <p>This is a simple UI page.</p>
</body>
</html>
```
3. Create a Dockerfile: Create a Dockerfile to containerize your app.
```
# Dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```
## Step 2: Push the Project to GitHub
1. Initialize Git Repository:
```
git init
git add .
git commit -m "Initial commit"
```
2. Create a GitHub Repository:
Go to GitHub, create a new repository, and follow the instructions to add the remote repository.
```
git remote add origin https://github.com/your-username/simple-ui-app.git
git push -u origin master
```
If any problem regarding pull, use this command:
```
git pull origin master --allow-unrelated-histories
```
## Step 3: Create a GitHub Actions Workflow
1. Create Workflow File: In your project, create a .github/workflows directory and add a workflow file named ci.yml.
```
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: your-dockerhub-username/simple-ui-app:latest

      - name: Logout from Docker Hub
        run: docker logout
```
2. Add Secrets to GitHub:
Go to your GitHub repository, click on Settings -> Secrets and variables -> Actions, and add the following secrets:
```
DOCKER_USERNAME: Your Docker Hub username.
DOCKER_PASSWORD: Your Docker Hub password or access token.
```
3. Verify Container Registry Access:
-Public Registry: If you are using a public container registry (like Docker Hub), make sure the image is publicly accessible and you’ve used the correct image name and tag.
-Private Registry: If you are using a private registry, ensure that Kubernetes has the correct credentials to access the registry. You might need to create a Kubernetes Secret with your registry credentials and reference it in your deployment.
To create a Docker registry secret, you can use the following command:
```
kubectl create secret docker-registry my-registry-key \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=<your-docker-username> \
  --docker-password=<your-docker-password> \
  --docker-email=<your-email>
```
Then, update your deployment to use this secret:
```
spec:
  containers:
  - name: mycontainer
    image: {{.Values.image.repository}}:{{.Values.image.tag}}
    ...
  imagePullSecrets:
  - name: my-registry-key
```
4. Commit and Push:
```
git add .github/workflows/ci.yml
git commit -m "Add GitHub Actions workflow"
git push
```
## Step 4: Deploy Using Minikube with Helm
1. Start Minikube:
```
minikube start
```
2. Create a Helm Chart:
```
helm create simple-ui-helm-chart
cd simple-ui-helm-chart
```
3. Edit the Helm Chart to Use Your Docker Image:
In the values.yaml file, set your Docker image:
```
# values.yaml
image:
  repository: your-dockerhub-username/simple-ui-app
  tag: latest
  pullPolicy: IfNotPresent
```
4. Deploy the Helm Chart:
```
helm install simple-ui ./simple-ui-helm-chart
```
5. Access Your Application:

If using Minikube, you can access the service by running:
```
minikube service simple-ui
```
