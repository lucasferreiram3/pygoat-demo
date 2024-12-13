pipeline {
    agent any 
    environment {
        caminhoPacote = 'uploadToVeracode/upload.tar.gz'
        wrapperVersion = '24.10.15.0'
        appProfile = 'pygoat-demo'
    }

    stages {
        stage('Archive') { 
            steps {
                sh 'mkdir uploadToVeracode'
                sh 'find "${WORKSPACE}" -name "*.py*" -o -name "*.html*" -o -name "*.js*" -o -name "*.css*" -o -name "*.json*" -o -name "*.lock*" | tar --exclude=./uploadToVeracode --exclude=./*.git* --exclude=./*.github*  --exclude=./*test* --exclude=./*mock* -cvzf uploadToVeracode/upload.tar.gz .'
            }
        }

       stage('Veracode SCA - Agent Scan') { 
            steps {
                withCredentials([string(credentialsId: 'SCA_TOKEN_PYGOAT', variable: 'SRCCLR_API_TOKEN')]) {
                    sh 'curl -sSL https://download.sourceclear.com/ci.sh | bash -s scan --update-advisor --uri-as-name || true'
                }
            }
        }

        stage('Veracode SAST - Sandbox Scan') { 
            steps {
                withCredentials([usernamePassword(credentialsId: 'veracode-credentials', passwordVariable: 'VKEY', usernameVariable: 'VID')]) {
                    sh 'curl -o veracode-wrapper.jar https://repo1.maven.org/maven2/com/veracode/vosp/api/wrappers/vosp-api-wrappers-java/${wrapperVersion}/vosp-api-wrappers-java-${wrapperVersion}.jar'
                    sh 'java -jar veracode-wrapper.jar -vid "${VID}" -vkey "${VKEY}" -action uploadandscan -appname ${appProfile} -createprofile true -filepath ${caminhoPacote} -createsandbox true -sandboxname "${BRANCH_NAME}" -deleteincompletescan 2 -version "${BUILD_NUMBER}" '
                }
            }
        }
        
        stage('Veracode SAST - Policy Scan') { 
            steps {
                withCredentials([usernamePassword(credentialsId: 'veracode-credentials', passwordVariable: 'VKEY', usernameVariable: 'VID')]) {
                    sh 'curl -o veracode-wrapper.jar https://repo1.maven.org/maven2/com/veracode/vosp/api/wrappers/vosp-api-wrappers-java/${wrapperVersion}/vosp-api-wrappers-java-${wrapperVersion}.jar'
                    sh 'java -jar veracode-wrapper.jar -vid "${VID}" -vkey "${VKEY}" -action uploadandscan -appname ${appProfile} -createprofile true -filepath ${caminhoPacote} -deleteincompletescan 2 -version "${BUILD_NUMBER}" '
                }
            }
        }

        stage('Veracode SAST - Pipeline Scan') { 
            steps {
                withCredentials([usernamePassword(credentialsId: 'veracode-credentials', passwordVariable: 'VKEY', usernameVariable: 'VID')]) {
                    sh 'curl -sSO https://downloads.veracode.com/securityscan/pipeline-scan-LATEST.zip'
                    sh 'unzip -o pipeline-scan-LATEST.zip'            
                    sh 'java -jar pipeline-scan.jar --veracode_api_id "${VID}" --veracode_api_key "${VKEY}" --file ${caminhoPacote} --issue_details true || true'
                }
            }
        }

        stage("Clean Workspace") {
            steps {
                script {
                    sh 'ls -l'
                    cleanWs()
                    sh 'ls -l'
                }
            }
        }
    }
}