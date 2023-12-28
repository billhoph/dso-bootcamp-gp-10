pipeline {
  agent any
  stages {
    stage('Git Checkout') {
      parallel {
        stage('Git Checkout') {
          steps {
            sh '''rm -Rf ./dso-bootcamp-gp-10

git clone https://github.com/billhoph/dso-bootcamp-gp-10.git
ls ./dso-bootcamp-gp-10'''
          }
        }

        stage('Clean up Images on Node') {
          steps {
            sh 'if docker image inspect harbor.alson.space/jenkins/yelb-ui:1.10 >/dev/null 2>&1; then docker rmi harbor.alson.space/jenkins/yelb-ui:1.10; fi'
            sh 'if docker image inspect harbor.alson.space/jenkins/yelb-db:1.10 >/dev/null 2>&1; then docker rmi harbor.alson.space/jenkins/yelb-db:1.10; fi'
            sh 'if docker image inspect harbor.alson.space/jenkins/yelb-appserver:1.10 >/dev/null 2>&1; then docker rmi harbor.alson.space/jenkins/yelb-appserver:1.10; fi'
          }
        }

      }
    }

    stage('Code Scanning') {
      steps {
        sh 'checkov -d ./dso-bootcamp-gp-10 --use-enforcement-rules --prisma-api-url https://api.sg.prismacloud.io --bc-api-key $pcskey --repo-id dso-bootcamp-gp-10/code-checking --branch main -o cli --quiet --compact'
      }
    }

    stage('Docker Image Building') {
      parallel {
        stage('Building UI app Image') {
          steps {
            sh 'docker build --no-cache ./dso-bootcamp-gp-10/yelb-ui -t harbor.alson.space/jenkins/yelb-ui:1.10'
          }
        }

        stage('Building DB Image') {
          steps {
            sh 'docker build --no-cache ./dso-bootcamp-gp-10/yelb-db -t harbor.alson.space/jenkins/yelb-db:1.10'
          }
        }

        stage('Building AppServer Image') {
          steps {
            sh 'docker build --no-cache ./dso-bootcamp-gp-10/yelb-appserver -t harbor.alson.space/jenkins/yelb-appserver:1.10'
          }
        }

      }
    }

    stage('List Successful Build Images') {
      steps {
        sh 'docker images'
      }
    }

    stage('Scanning Images (UI Image)') {
      parallel {
        stage('Scanning Images (UI Image)') {
          steps {
            prismaCloudScanImage(image: 'harbor.alson.space/jenkins/yelb-ui:1.10', resultsFile: './yelb-ui-10-scan.json', logLevel: 'info', dockerAddress: 'unix:///var/run/docker.sock')
          }
        }

        stage('Scanning Images (App Image)') {
          steps {
            prismaCloudScanImage(image: 'harbor.alson.space/jenkins/yelb-appserver:1.10', resultsFile: './yelb-app-10-scan.json', logLevel: 'info', dockerAddress: 'unix:///var/run/docker.sock')
          }
        }

        stage('Scanning Images (DB Image)') {
          steps {
            prismaCloudScanImage(dockerAddress: 'unix:///var/run/docker.sock', image: 'harbor.alson.space/jenkins/yelb-db:1.10', resultsFile: './yelb-db-10-scan.json', logLevel: 'info')
          }
        }

      }
    }

    stage('Publish Scan Result') {
      steps {
        sh 'ls -tlr'
        prismaCloudPublish(resultsFilePattern: '*.json')
      }
    }

    stage('Push Images to Harbor') {
      steps {
        sh '''docker login harbor.alson.space -u admin -p Harbor12345
docker push harbor.alson.space/jenkins/yelb-ui:1.10
docker push harbor.alson.space/jenkins/yelb-db:1.10
docker push harbor.alson.space/jenkins/yelb-appserver:1.10
'''
      }
    }

    stage('Deploy Application') {
      steps {
        sh '''kubectl cluster-info --context kind-demo
kubectl get pod -A
kubectl delete ns dso-bootcamp-gp-10
kubectl create ns dso-bootcamp-gp-10
kubectl apply -f ./dso-bootcamp-gp-10/deployments/platformdeployment/Kubernetes/yaml/yelb-k8s-minikube-nodeport.yaml -n dso-bootcamp-gp-10
kubectl get svc/yelb-ui -n dso-bootcamp-gp-10'''
      }
    }

  }
  environment {
    pcskey = '1c788e25-83ba-4b31-94d6-3f99234e2e46::DdgP+op52pyjTlAF+nnpkXxt0i4='
  }
}