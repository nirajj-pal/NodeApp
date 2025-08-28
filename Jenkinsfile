pipeline {
  agent any

  tools { nodejs 'NodeJS' }

  environment {
    REGISTRY = '10.48.228.71'
    HARBOR_PROJECT = 'hgtspl'       // Harbor project
    IMAGE = 'nodeapp'               // repository under the project
    HARBOR_CREDENTIALS_ID = 'harbor-credentials'
    PATH = "${WORKSPACE}/.bin:${PATH}"   // kubectl lives in workspace
  }

  stages {
    stage('Checkout Github') {
      steps {
        git branch: 'main', credentialsId: 'jen-doc-git', url: 'https://github.com/nirajj-pal/NodeApp.git'
      }
    }

    stage('Install node dependencies') {
      steps { sh 'npm install' }
    }

    stage('Test Code') {
      steps { sh 'npm test' }  // exits 0; change package.json when you add real tests
    }

    stage('Build Docker Image') {
      steps {
        sh '''#!/usr/bin/env bash
set -euo pipefail
export DOCKER_BUILDKIT=1

FULL_IMAGE="${REGISTRY}/${HARBOR_PROJECT}/${IMAGE}"
echo "Docker version:"; docker --version
echo "Building ${FULL_IMAGE}:{latest,${BUILD_NUMBER}}"

docker build \
  -t "${FULL_IMAGE}:latest" \
  -t "${FULL_IMAGE}:${BUILD_NUMBER}" \
  .
'''
      }
    }

    stage('Push Image to Harbor') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${HARBOR_CREDENTIALS_ID}",
                                          usernameVariable: 'HUSER',
                                          passwordVariable: 'HPASS')]) {
          sh '''#!/usr/bin/env bash
set -euo pipefail
FULL_IMAGE="${REGISTRY}/${HARBOR_PROJECT}/${IMAGE}"

echo "$HPASS" | docker login "${REGISTRY}" -u "$HUSER" --password-stdin
docker push "${FULL_IMAGE}:${BUILD_NUMBER}"
docker push "${FULL_IMAGE}:latest"
'''
        }
      }
    }

    stage('Install Kubectl') {
      steps {
        sh '''#!/usr/bin/env bash
set -euo pipefail
mkdir -p .bin
curl -sSL -o .bin/kubectl "https://dl.k8s.io/release/$(curl -sSL https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x .bin/kubectl
.bin/kubectl version --client
'''
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        script {
          kubeconfig(
            caCertificate: 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJZEl3VGJVcCtKRHN3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TlRBNE1qVXhNVEUwTWpkYUZ3MHpOVEE0TWpNeE1URTVNamRhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUURqc0QxUVphbjZaMUlxZVNYNlYwcWp0MUN5ZkpTaUVPcGpvUzl5QjYvVGdYeW8xQ1lTMVUzLytrSWUKenZkbzVRUjNKSDJKaW1CblFIUXVUbWQzb0lTdElnUGdZbzVpNU5uL2VtOURvcjd0ZTcrc0xaVXZ3eEU4YkNJZwo5bUxTMml2TWhndC84ajlrVlAzNEFFdE9ua1c4aW9Da0VaSXVLL01xeklNVlVsWXMzTU5EbVVvQmhmQm42enh5CnRjUDRuRU9kOFp6YUtvMWRJcHExN0RoRWZEbU5HVGZSTWp5QUs4OEtWVzF1Slg1UXVBQTJiSTBEeVVkZzhvdmEKWFQ0TThLdms3WnM0NmN4V0ZLQmx2RVpTcmFDVXRMMFVuTTBhWFl4RlJNb1JtZHZDTFdpUXkrZGRwNXpiOXNjaAppYlh3Szlrbi9VYU45Mk91UjZ4bzVTNFFHU0Y1QWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJRRHd5eGhpMGRLbXV1VkNUTURscWg2dGlYRVBqQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQ3Y2cTdGNkw5dQpDaVBZZU1yYzdZZEZ1VGNnMWVQbXoyZkREcTNKZmJLajNCTGJTd3ZoL1pRMjh4cVV5clh2anM5NDVLbDdmc0JKCkMva2gzdEV4MldXUk9xbFdieVZ2UTlDYkV4T3dDTTkzS2VjUEsvdjl6S1NaU3VsdGh4VitJSGdTaVNheGs1SVAKRnJJME0xbjFYaWhDVjJ3RUMzck42NWZlMEphTjlRc2tFUVV2TG9VSk0vM0lpOFdsVTdGZFJFemwwTFVCZ2FDegpWTUc3RDdNZFBScHJuMk5LWWhMNnRmeUlxMlNpSE9rSU1VRk5zYlFZQ2p1VEJ5QUZhYjk0b0JiM2Y3d3M1dHpFClBQME1OcXpjZmJ0SUZUQkZTa1JnTHNNRTg5KzhPVWxtU2taSW96cVhLNFNCbElqcFlnb0w2UnRTK0ZWeERVS3cKNElMS05RY0JFa1lqCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K',
            credentialsId: 'kubeconfig',
            serverUrl: 'https://10.48.228.66:6443'
          ) {
            sh 'kubectl apply -f deployment.yaml'
          }
        }
      }
    }
  }

  post {
    success { echo 'Build&Deploy completed succesfully!' }
    failure { echo 'Build&Deploy failed. Check logs.' }
  }
}
