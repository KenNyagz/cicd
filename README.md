# DevOps Capstone

Flask app, tested and containerised with Docker, built and deployed by Jenkins.

## Run locally
    docker build --target runtime -t devops-capstone .
    docker run -p 5000:5000 devops-capstone
    curl localhost:5000/health

## Pipeline
Checkout -> Test -> Build Image -> Push Image -> Deploy -> Smoke Test

## Jenkins setup notes
Run Jenkins with access to the host Docker daemon:

    docker run -d --name jenkins -p 8080:8080 -p 50000:50000 \
      -v jenkins_home:/var/jenkins_home \
      -v /var/run/docker.sock:/var/run/docker.sock \
      -u root jenkins/jenkins:lts

    docker exec jenkins sh -c "apt-get update && apt-get install -y docker.io"

Add a Docker Hub credential with ID `dockerhub-creds`.
Create a Multibranch Pipeline job pointing at your GitHub repo.
