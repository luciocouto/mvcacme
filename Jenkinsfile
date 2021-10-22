#!groovy

def scmVars

def additionalProperties = [
    parameters([
        booleanParam(defaultValue: true, description: 'Executar análise sonar?', name: 'executaAnaliseSonar'),
    ])
]

//PROPERTIES
propertiesGlobalSetup(vertical, productName, projectName, additionalProperties)
try {
  podTemplate(
        label: label,
        containers: [
            containerTemplate(
                name: 'dotnet',
                image: 'mcr.microsoft.com/dotnet/sdk:3.1',
                alwaysPullImage: false,
                command: 'sh',
                ttyEnabled: true,
                privileged: true),
            containerTemplateDocker('docker'),
        ],
        imagePullSecrets: getPodTemplateImagePullSecrets(),
        volumes: getPodTemplateVolumes()
    ) {
        node(label) {
            container('dotnet') {
                try{
                    // Efetua o chackout da solution
                    stage('Checkout') {
                        scmVars = checkout scm
                    }
                    // Instala Java 11 para o sonar scanner
                    stage('Install Java 11') {
                        if(params.executaAnaliseSonar){
                            echo 'Execução sonar habilitada'
                            echo 'Necessário instalar Java'

                            sh 'apt update'
                            sh 'apt install openjdk-11-jre-headless -y'
                        }
                        else {
                            echo 'Execução sonar desabilitada'
                            echo 'Não é necessário instalar Java'
                        }
                    }
                     stage('SCM') {
                                    git 'https://github.com/foo/bar.git'
                                  }

                    // Inicia analise para sonarQube
                    stage('SonarQube Begin') {
                        if(params.executaAnaliseSonar){
                            echo 'Execução sonar habilitada'

                            echo 'Restaurar dotnet-sonarscanner'
                            sh 'dotnet tool restore'

                            def SONAR_PROJECT_KEY= 'TesteJenkins'
                            def SONAR_URL= 'http://localhost:19000'
                            
                            withSonarQubeEnv('http://localhost:19000') {

                                def cmd = "dotnet tool run dotnet-sonarscanner begin"
                                cmd += " /k:\"${SONAR_PROJECT_KEY}\""
                                cmd += ' /d:sonar.qualitygate.wait=true'

                                echo 'Execução inicio sonar'
                                sh cmd
                            }
                        } else {
                            echo 'Execução sonar desabilitada'
                        }
                    }
                    //Executa o build observando o ambiente: dev, hmg ou pre
                    stage('Build') {
                        echo 'Compilando solution'
                        sh "dotnet build -v d -c ${conf}"
                    }                  
                    // Finaliza analise para sonarqube e envia resultados
                    stage('SonarQube end') {
                        if(params.executaAnaliseSonar) {
                             withSonarQubeEnv('http://localhost:19000') {
                                def cmd = "dotnet tool run dotnet-sonarscanner end"

                                echo 'Execução fim sonar'
                                sh cmd
                            }
                        }
                        else {
                            echo 'Execução sonar desabilitada'
                        }
                    }
                }
                finally {
                    cleanWs()
                }
            }
        }
    }
}
catch (e) {
  currentBuild.result = 'FAILURE'
  throw e
}
finally {
    onFinally(vertical, productName, projectName )