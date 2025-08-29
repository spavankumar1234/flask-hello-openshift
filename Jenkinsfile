pipeline {
    agent any

    environment {
        IMAGE_NAME = "flask-hello-openshift"
        REGISTRY   = "image-registry.openshift-image-registry.svc:5000"
        PROJECT    = "jenkins-project"   // change if you want a new namespace
    }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/spavankumar1234/flask-hello-openshift.git'
            }
        }

        stage('Build App') {
            steps {
                sh 'echo "Building Flask App..."'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'echo "Running dummy tests..."'
            }
        }

        stage('Build & Push Image') {
            steps {
                sh '''
                oc login --insecure-skip-tls-verify=true \
                         --token=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token) \
                         --server=https://kubernetes.default.svc
                oc project $PROJECT
                oc start-build $IMAGE_NAME --from-dir=. --wait=true
                '''
            }
        }
stage('Deploy App') {
    steps {
        sh '''
        oc login --insecure-skip-tls-verify=true \
                 --token=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token) \
                 --server=https://kubernetes.default.svc
        oc project $PROJECT

        cat <<EOF | oc apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: $IMAGE_NAME
  labels:
    app: $IMAGE_NAME
spec:
  replicas: 1
  selector:
    matchLabels:
      app: $IMAGE_NAME
  template:
    metadata:
      labels:
        app: $IMAGE_NAME
    spec:
      containers:
      - name: $IMAGE_NAME
        image: $REGISTRY/$PROJECT/$IMAGE_NAME:latest
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: $IMAGE_NAME
  labels:
    app: $IMAGE_NAME
spec:
  selector:
    app: $IMAGE_NAME
  ports:
  - port: 80
    targetPort: 8080
EOF
        '''
    }
}

stage('Expose Route') {
    steps {
        sh '''
        oc expose svc/$IMAGE_NAME || true
        '''
    }
}

