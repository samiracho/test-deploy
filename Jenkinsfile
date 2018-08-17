node {
    def mvnHome = tool 'M3'
    def projectName = 'sandbox-000000'
    def appName = 'apisamplejava'
    def dockerImageTag  = "eu.gcr.io/${projectName}/${appName}:${profile}-${image_tag}"
    def build = ( "${action}" == "build and deploy" || "${action}" == "only build") ? "true" : "false"
    def deploy = ( "${action}" == "build and deploy" || "${action}" == "only deploy") ? "true" : "false"
    def commitHash = ""
    def scriptFolder = "${WORKSPACE}@script"
    def k8sFilePath = "scriptFolder/K8sfile"
   
    stage('Checkout code') {
      git branch: "${branch}", url: "${repository}"
      commitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
    }
   
    if ("${build}" == "true") {
        
        stage ('Code Analysis') {
            // We execute the different code analysis tools
            sh "${mvnHome}/bin/mvn -batch-mode -V -U -e findbugs:findbugs"
            findbugs canComputeNew: false, defaultEncoding: '', excludePattern: '', healthy: '', includePattern: '', pattern: '**/target/findbugsXml.xml', unHealthy: ''
        }
       
        stage('Build Artifact') {
            sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
        }
       
        stage ('Build Docker image') {
            sh "cp ${scriptFolder}/Dockerfile ."
            sh "docker build -t ${dockerImageTag} ."
        }
        
        stage ('Test Docker Image') {
            dir(scriptFolder){     
                sh "dgoss run ${dockerImageTag}"
            }
        }
        
        stage ('Push Docker image') {
            sh "docker push ${dockerImageTag}"
        }
    }
    
    if ("${deploy}" == "true") {

        stage ('Deploy in Kubernetes') {
            def currentDate = sh(script: "date +'%s'", returnStdout: true).toString().trim()
            def currentImage = sh(script: "kubectl get deployment ${appName} -o=jsonpath=\'{\$.spec.template.spec.containers[:1].image}\' --namespace=${profile} || true", returnStdout: true).toString().trim()
            sh("kubectl get ns ${profile} || kubectl create ns ${profile}")
            sh("sed -i.bak 's#IMAGE_PLACEHOLDER#${dockerImageTag}#' ${k8sFilePath}")
            sh("sed -i.bak 's#COMMIT_PLACEHOLDER#${commitHash}#' ${k8sFilePath}")
            sh("kubectl --namespace=${profile} apply -f ${k8sFilePath}")               
            if("${currentImage}" == "${dockerImageTag}" ){
                sh("kubectl patch deployment ${appName} -p \'{\"spec\":{\"template\":{\"metadata\":{\"labels\":{\"date\":\"${currentDate}\"}}}}}\' --namespace=${profile}")
            }
        }
    }
}
   
