# CI/CD Pipeline with Docker, Kubernetes, Jenkins, and SonarQube

A step-by-step guide to setting up a CI/CD pipeline for a Java application using Docker, Kubernetes, Jenkins, and SonarQube.

---

## 🎯 Assignment Objective

This project follows a complete CI/CD workflow:

1. **Checkout Code** → Pull the latest Java code from GitHub  
2. **Build the Java Application** → Compile using **Java 17**  
3. **Run Unit Tests** → Test using **Java 11**  
4. **Analyze Code Quality** → Use **SonarQube** and **Java 8**  
5. **Build Docker Image** → Package the Java application inside a Docker container  
6. **Push to Docker Registry** → Store the image in **Docker Hub**  
7. **Deploy to Kubernetes** → Deploy the containerized application using Kubernetes manifests

---

## 🐳 Install Docker Desktop

1. Download Docker: [docker.com/get-started](https://www.docker.com/get-started/)
2. Choose Windows (AMD64), install with default settings.
3. Accept the Docker subscription agreement, then finish setup.
4. Restart if prompted.
5. Run Docker Desktop and check it's running via:
   ```bash
   docker version
   ```
   - `Client:` shows your Docker version
   - `Server:` confirms Docker engine is up

6. If needed, install WSL (Windows Subsystem for Linux):
   - Search for "WSL" in Start, then install

---

## 🧑‍💻 Install Git & Setup GitHub

1. Download Git: [git-scm.com](https://git-scm.com/downloads/win)
2. Install with default settings
3. Set global config:
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "your-email@example.com"
   ```

4. Generate SSH key:
   ```bash
   ssh-keygen -t ed25519 -C "your-email@example.com"
   ```
   - Leave everything blank when prompted
   - Get the public key:
     ```bash
     type C:\Users\<your-user>\.ssh\id_ed25519.pub
     ```
5. Go to GitHub > Settings > SSH and GPG keys > Add New SSH Key
6. Test connection:
   ```bash
   ssh -T git@github.com
   ```

---

## 💡 Install IntelliJ (or Eclipse)

- [Download IntelliJ](https://www.jetbrains.com/idea/download/?section=windows)
- Use your university account for free access
- Install with default settings
- Restart when prompted

---

## ☕ Build Java App (Maven)

1. Create a Maven project in IntelliJ
2. Ensure JDK 17 is installed (Ctrl + Alt + Shift + S → SDKs)
3. Write your app code:
   - `Calculator.java`
   - `CalculatorTest.java`
   - `pom.xml` for Maven build

4. Run the app & tests locally to verify

---

## 🐋 Dockerize the Java App

1. Create a `Dockerfile` in your project
2. Build and run your image:
   ```bash
   docker build -t my-java-app .
   docker run my-java-app
   ```

---

## 🛠 Jenkins Setup in Docker

1. Create a custom Docker network:
   ```bash
   docker network create ci_network
   ```

2. Run Jenkins:
   ```bash
   docker run -d --name jenkins -p 8080:8080 -p 50000:50000 \
     --network ci_network -v jenkins_home:/var/jenkins_home \
     jenkins/jenkins:lts
   ```

3. Access Jenkins at `localhost:8080`
4. Use the container logs to find the initial password:
   ```bash
   docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   ```

5. Install suggested plugins, and also manually install:
   - Pipeline
   - Docker Pipeline
   - SonarQube Scanner
   - Kubernetes

---

## 📊 Run SonarQube

1. Start SonarQube container:
   ```bash
   docker run -d --name sonar -p 9000:9000 --network ci_network sonarqube:lts
   ```

2. Access it at `localhost:9000`
3. Login with `admin:admin` and update the password
4. Create a project and generate a token

---

## 📦 Java Docker Containers

Create Java containers (for assignment requirement):

```bash
docker run -dit --name java8 --network ci_network openjdk:8-jdk bash
```

---

## 🧪 Jenkins Pipeline Setup

1. Push Java project to GitHub:
   ```bash
   git init
   git remote add origin git@github.com:YourUsername/Pipeline-with-Docker.git
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git push -u origin main
   ```

2. Create `Jenkinsfile` in your project root and push it to GitHub

3. In Jenkins:
   - Set up Maven under Global Tools (same name as used in `Jenkinsfile`)
   - Add SonarQube credentials
   - Add DockerHub credentials
   - Add GitHub SSH private key

4. Create a new pipeline job in Jenkins:
   - Name: `Pipeline-Docker-Java-App`
   - Pipeline from SCM (Git), branch: `main`
   - Script path: `Jenkinsfile`

5. Build the job – it should pass!

---

## ☸️ Kubernetes Setup

1. Enable Kubernetes from Docker Desktop > Settings
2. Apply and restart
3. Verify Kubernetes is running:
   ```bash
   kubectl cluster-info
   ```

4. Create a `deployment.yaml` and push it to GitHub
5. Copy your kube config file into Jenkins container:
   ```bash
   docker cp ~/.kube/config jenkins:/root/.kube/config
   ```

6. Install `kubectl` inside Jenkins container if needed

---

## ✅ Final Result

- Java app built & tested
- Docker image created
- Code analyzed with SonarQube
- Pipeline deployed the app into Kubernetes

You now have a working CI/CD pipeline! 🎉
