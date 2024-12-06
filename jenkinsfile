pipeline {
    agent any
    parameters {
        string(name: 'CLIENT_NAME', description: 'Client name (e.g., client1)')
        string(name: 'PORT', description: 'Port for Ghost service (e.g., 2368)')
        string(name: 'DB_PASSWORD', description: 'Database password for the client')
    }
    stages {
        stage('Build Deployment YAML') {
            steps {
                script {
                    // Define the Kubernetes Deployment and Service YAML content dynamically
                    def deploymentYaml = """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ghost-${params.CLIENT_NAME}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ghost
      client: ${params.CLIENT_NAME}
  template:
    metadata:
      labels:
        app: ghost
        client: ${params.CLIENT_NAME}
    spec:
      containers:
      - name: ghost
        image: ghost:latest  # Using the prebuilt Ghost image
        ports:
        - containerPort: 2368
        env:
        - name: url
          value: http://10.1.1.30:${params.PORT}
        - name: database__connection__password
          value: ${params.DB_PASSWORD}
---
apiVersion: v1
kind: Service
metadata:
  name: ghost-${params.CLIENT_NAME}
spec:
  type: NodePort
  ports:
  - port: 2368
    targetPort: 2368
    nodePort: ${params.PORT}  # Use the dynamic port parameter here
  selector:
    app: ghost
    client: ${params.CLIENT_NAME}
                    """
                    // Write the generated YAML to a file
                    writeFile file: 'deployment.yaml', text: deploymentYaml

                    // Optional: Debug to check generated content
                    sh 'cat deployment.yaml'
                }
            }
        }
        stage('Apply Kubernetes Configuration') {
            steps {
                script {
                    // Deploy the generated Kubernetes resources to k3s
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }
}
