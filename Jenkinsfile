
pipeline{
    //any is good for when you have clusters, running on the next available agent
    agent any
​
    environment{
        VERACODE_APP_NAME = "reactvulna"
      //  DBG = true
 //       HOST_OS = checkOs()
        APIVERSION = "23.4.11.2"
​
        // STAGES = ["Development", "Feature", "Staging", "Main"]
        DEVELOPMENT_STAGE = "Development"
        //checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/veracode/verademo']]])
    }
​
    // tools{
    //     maven "Maven-3"
    // }
​
    stages{
        stage("Enviornment Set Up"){
            steps{
​
                // Testing Dev Stages
                echo "Development Stage: ${DEVELOPMENT_STAGE}"
​
                //################################################################################################
                // Experimental:
                echo '============================ Determine the operating system ============================'
                echo 'The pipeline is being built in: '
      //          echo checkOs()
      //          script{
     //               env.HOST_OS = checkOs()
                }//end script
                //################################################################################################
​
 //               git branch: 'master', url: 'https://github.com/mokkang/NodeGoat'
​
​
                //################################################################################################
                //  Setting up Pipeline Scanner Enviornment
                //################################################################################################
                // Requirements:
                // https://docs.veracode.com/r/About_Pipeline_Scan_Prerequisites
                //      - Java 8 or later
                //      - Enable port 443 in the enviornment using standard HTTPS
                //      - API credentials for a service account or account created
                //          - Permissions:
                //              - Uplooad and Scan API to create application profiles and upload scan applicaitons
                //              - Upload API - Submit Only to submit scans
                //      Note: A veracode account is limited to six pipeline scans per 60 seconds and each scan is limited to a max scan time of 60 minutes
                //
                // For the application you want to scan:
                //      -
                //
                //
                //
                //
​
​
            }
        }
        stage("Build"){
            
            steps{
                
                //
                //
                //
                //
                //
                echo '============================ Building the application ============================'
                script{
                    if(isUnix() == true){
​
                        //########################################################################################
                        // Moves into build location directory, location of build files
                        // Building the application using maven on a Windows Jenkins Host
                        //########################################################################################    zip -r nodegoat-veracode.zip -x NodeGoat/node_modules/\* NodeGoat/.\*/\*
                        powershell '''
                            ls
                            npm install
                            Compress-Archive . NodeGoat.zip
                            ls
                        '''
                    }
                    else{
​
                        //########################################################################################
                        // Moves into build location directory, location of build files
                        // Building the application using maven on a linux/Unix Jenkins Host
                        //########################################################################################
                        
                        sh "npm install ."
                        sh "zip -r NodeGoat.zip  -x NodeGoat/node_modules/"
                        echo '===================== Checking directorty after build ====================='
                        sh 'ls'
                        sh 'pwd'
                    }
                }
                
                
                
                echo '============================ Ending the build stage ============================'
            }
        
        }
        
        // ##########################################################################################################
        //
        //      Veracode Software Composition Analysis Agent-Based Scan
        //
        // ##########################################################################################################
​
​
        stage("SCA scan"){
            steps{
                script{
​
​
                    // ##############################################################################################
                    //  Within the location of the build files, using the withCredentials to se the API token as an 
                    //      enviornmental variable within the scope of the stage. This can be done globally as well
                    // ##############################################################################################
​
                   
                        
                        echo '===================== Checking directorty contents in build location ====================='
                        powershell 'ls'
                        
                        withCredentials([string(credentialsId: 'SCA_Token', variable: 'SRCCLR_API_TOKEN')]) {
                            if(isUnix() == true){
                                
                                // ###################################################################################
                                // Veracode SCA Agent-based scan CI installation for Linux/Unix based Jenkins Host
                                // ###################################################################################
​
                                sh 'curl -sSL https://download.sourceclear.com/ci.sh | sh'
                                //sh "curl -sSL https://download.sourceclear.com/ci.sh | DEBUG=1 sh -s -- scan --no-upload"
                            }//end if
                            else{
​
                                // ###################################################################################
                                // Veracode SCA Agent-based scan CI installation for Windows based Jenkins Host
                                // ###################################################################################
​
                                powershell 'ls'
                                powershell "iex ((New-Object System.Net.WebClient).DownloadString('https://download.srcclr.com/ci.ps1')); srcclr scan "
                            }//end else
                        }//end withCredentials
                   
               
                }//end script
            }//end steps
        }//end stage
        
        
        // ##########################################################################################################
        //
        //      Veracode ( Static Analysis Agent-Based ) Pipeline Scan ( Findings not sent to the platform )
        //          This scan is used for quick scans between development commits, you cna utilize a baseline scan to
        //          find net new flaws that were not identified in the baseline file, as well as scan against a policy
        //          which can be pulled down from the platform.
        //
        // ##########################################################################################################
​
        stage("pipeline scan"){
​
            steps{
                echo '============================ Downloading Pipeline Files =================================='
                script{
                    if(isUnix() == false){
                        bat 'curl https://downloads.veracode.com/securityscan/pipeline-scan-LATEST.zip -o pipeline-scan.zip'
                        powershell 'Set-ExecutionPolicy AllSigned -Scope Process -Force'
                        
                        // Experimental
                        bat 'curl https://repo1.maven.org/maven2/com/veracode/vosp/api/wrappers/vosp-api-wrappers-java/23.4.11.2/vosp-api-wrappers-java-23.4.11.2-dist.zip -o ApiWrapper.zip '
                        powershell 'Expand-Archive -Force -Path ApiWrapper.zip -DestinationPath ApiWrapper'
                        powershell 'ls'
                        
                    }
                    else{
                        sh 'curl https://downloads.veracode.com/securityscan/pipeline-scan-LATEST.zip -o pipeline-scan.zip'
                        
                    }
                }
                powershell 'Expand-Archive -Force -Path pipeline-scan.zip -DestinationPath pipeline-scanner '
                echo '============================= Unzipping Files ============================================'
                unzip test: true, zipFile: 'pipeline-scan.zip'
                
​
                echo '============================ Finishing environment checkout stage ============================'
​
​
                //unzip test: true, zipFile: 'dist.zip'
                echo '============================ Scanning the application ============================'
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    script{
​
                        echo '===================== Checking directorty contents after build ====================='
                        powershell 'ls'
​
​
                        if(isUnix() == false){ 
                            //windows
                            echo '---------------------------- Pipeline scan starting - Windows based -----------------------------'
                            withCredentials([usernamePassword(credentialsId: 'veracode_login', passwordVariable: 'veracode_api_key', usernameVariable: 'veracode_api_id')]) {
                                powershell '''java -jar pipeline-scanner/pipeline-scan.jar -f reactvulna.zip --veracode_api_id "$env:veracode_api_id" -vkey "$env:veracode_api_key"  -id true --fail_on_cwe="80" --fail_on_severity="Very High, High"  '''
                            }
                        }
                        else{
                            //linux
                            echo '---------------------------- Pipeline scan starting - Unix based -----------------------------'
                            withCredentials([usernamePassword(credentialsId: 'veracode_login', passwordVariable: 'veracode_api_key', usernameVariable: 'veracode_api_id')]) {
                                sh '''java -jar pipeline-scanner/pipeline-scan.jar -f reactvulna.zip --veracode_api_id "$env:veracode_api_id" -vkey "$env:veracode_api_key"  -id true --fail_on_cwe="80" --fail_on_severity="Very High, High" || true''' 
                            }
                        }
                    }
                }
                
            }
        }
​
        // ##########################################################################################################
        //
        //      Veracode Static Analysis Upload and Scan
        //
        // ##########################################################################################################
​
​
​
        stage("Veracode Upload and Scan"){
            steps{
                script{
                    
                    withCredentials([usernamePassword(credentialsId: 'veracode_login', passwordVariable: 'VERACODE_API_KEY', usernameVariable: 'VERACODE_API_ID')]) {
                        veracode applicationName: 'reactvulna', criticality: 'VeryHigh', debug: true, fileNamePattern: '', replacementPattern: '', sandboxName: DEVELOPMENT_STAGE , scanExcludesPattern: '', scanIncludesPattern: '', scanName: BUILD_TAG, teams: 'Demo Team', uploadIncludesPattern: 'reactvulna.zip', vid: VERACODE_API_ID, vkey: VERACODE_API_KEY
                    }
​
                }
            }
        }
​
​
​
        // ##########################################################################################################
        //
        //      Veracode Dynamic Scan
        //
        // ##########################################################################################################
​
​
        
    }
    post{
        always{
            //things that are always run
          
            
            echo '============================ finished pipeline ============================'
        }
        failure{
            //things that run on failure
            echo '============================  failed  ============================'
        }
        success{
            //things that run on success
            echo '============================  Success  ============================'
        }
    }
}
