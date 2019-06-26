def label = "kubectl-${UUID.randomUUID().toString()}"



podTemplate(label: label, yaml: """
apiVersion: v1
kind: Pod
spec:
  containers:

  - name: helm
    image: dtzar/helm-kubectl:2.13.0
    command: ['cat']
    tty: true
  - name: docker
    image: docker:1.11
    command: ['cat']
    tty: true
    volumeMounts:
    - name: dockersock
      mountPath: /var/run/docker.sock
  volumes:
  - name: dockersock
    hostPath:
      path: /var/run/docker.sock
"""
  ){
  node(label) {
    stage('Checkout'){
        checkout scm
    }
    stage('Build') {
        container('docker') {
            withDockerRegistry([credentialsId: 'docker_registry_credentials']){
                def customImage = docker.build("${IMAGE_NAME}")
                customImage.push("${env.BUILD_NUMBER}")
                customImage.push("latest")
            }
        }
    }

    stage('Deploy') {
         container('helm') {

            withCredentials([file(credentialsId: "kube-config-${TARGET_CLUSTER}", variable: 'KUBECONFIG')]) {
                  sh "helm upgrade --install --set image.repositoryAndTag=${IMAGE_NAME}:latest hello-world ./helloworld-chart "
            }
        }
    }
  }
}



