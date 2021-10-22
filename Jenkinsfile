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
        node(main) {
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

                    // Inicia analise para sonarQube
                    stage('SonarQube Begin') {
                        if(params.executaAnaliseSonar){
                            echo 'Execução sonar habilitada'

                            echo 'Restaurar dotnet-sonarscanner'
                            sh 'dotnet tool restore'

                            def SONAR_PROJECT_KEY= 'TesteJenkins'
                            def SONAR_URL= 'http://localhost:29000'
                            
                            withSonarQubeEnv('http://localhost:29000') {

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
                             withSonarQubeEnv('http://localhost:29000') {
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
catch (e) {
  currentBuild.result = 'FAILURE'
  throw e
}
