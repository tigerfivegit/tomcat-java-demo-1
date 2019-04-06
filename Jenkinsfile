def label = "test-pod"
podTemplate(
  label: label, 
  cloud: 'kubernetes', 
  containers: [
    containerTemplate(name: 'docker', image: 'docker:18', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'maven', image: 'maven:3.3.9-jdk-8-alpine', ttyEnabled: true, command: 'cat'),
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', 
    hostPath: '/var/run/docker.sock'),
  ],
) 
{
  node(label) {
    stage('Checkout SCM') {
      git branch: 'master', url: 'https://github.com/jenkinsci/kubernetes-plugin.git'
      // 生成镜像标签，格式：commit号-更新时间
      tag = sh(returnStdout: true, script: "git rev-parse --short HEAD|tr -d '\n';echo -|tr -d '\n';date +%Y%m%d%H%M")
      project = "blog"
      app_name = "demo"
      namespace = "default"
      registry = "192.168.31.61/${project}"
      image_name = "${registry}/${app_name}:$tag"
      container('maven') {
          stage('Maven Build') {
              sh 'mvn clean install -DskipTests'
          }
      }
      container('docker') {
          stage('Build Docker Image') {
            sh '''
            cat pw.txt | docker login --username lizhenliang --password-stdin $registry && \
            docker build -t ${image_name} -f Dockerfile . && \
            docker push ${image_name}
            '''
          }
      }
    }
  }
}