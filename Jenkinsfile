pipeline{
    
    agent any
    
    environment{
        
        DEV_SERVER_IP = '65.2.186.98';
        SELENIUM_SERVER = '65.2.132.29';
        QA_SERVER_IP = '52.66.9.98';

    }
    
    
    stages{
        stage('Dev Deployment'){
            steps{
               sshagent(['ssh-dev-credentials']){
                   
                   sh '''
                   ssh -o StrictHostKeyChecking=no ubuntu@${DEV_SERVER_IP} '
                   
                        sudo fuser -k 80/tcp
                   
                        git clone https://github.com/TestLeafInc/jenkins-pipeline
                        
                        cd /home/ubuntu/jenkins-pipeline/leafhub
                        
                        git pull 
                        
                        // Get the commit id --> store it !
                        // Alternate appach: Make your build with the build number 
                        // And move to the nexus repo --> Ask your next server pick that
                        export COMMIT_ID = $(git rev-parse HEAD)
                        
                        nohup sudo mvn spring-boot:run > /dev/null 2>&1 & 
                        
                   
                   '
                   '''
               }
               
               
               
            }
            
            
            
        }
        
        stage('Smoke Tests against Dev'){
           
            
            steps{
                
                 // Get call to the Server URL --> 200 
                 sleep(10)
                    
                 sshagent(['ssh-dev-credentials']){
                     
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@${SELENIUM_SERVER} '
                    
                        # Update the package lists for upgrades and new package installations
                        sudo apt-get update
                        
                        # Install OpenJDK 8
                        sudo apt-get install -y openjdk-8-jdk
                        
                        # Install Maven
                        sudo apt-get install -y maven
                        
                        # Install xvfb
                        sudo apt-get install -y xvfb
                        
                        # Start xvfb
                        Xvfb :99 &
                        export DISPLAY=:99
                        
                        # Download and install Google Chrome
                        if ! command -v google-chrome > /dev/null; then
                          wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
                          sudo dpkg -i google-chrome-stable_current_amd64.deb
                          sudo apt --fix-broken install -y
                        else
                          echo "Google Chrome is already installed."
                        fi
                        
                        
                        # Clone the webdriver-tests repository
                        git clone https://github.com/TestLeafInc/webdriver-leafhub
                        
                        # Move to the webdriver-tests folder
                        cd webdriver-leafhub
                        
                        # pull the changes if any
                        git pull
                        
                        # Run Maven tests
                        mvn clean test -DsuiteXmlFile=smoke.xml -Dserver.ip=65.2.186.98
                        
                        # Push the results to S3 (make sure to install and configure awsconfigure before)
                        #aws s3 sync reports/ s3://reports-html-selenium
                        
                    
                    '
                    '''
                 }
            }
            
        }
        
        stage('QA Deployment'){
            when{
                expression {currentBuild.resultIsBetterOrEqualTo('SUCCESS')}
            }
            steps{
               sshagent(['ssh-dev-credentials']){
                   
                   sh '''
                   ssh -o StrictHostKeyChecking=no ubuntu@${QA_SERVER_IP} '
                   
                        sudo fuser -k 80/tcp
                   
                        git clone https://github.com/TestLeafInc/jenkins-pipeline
                        
                        cd /home/ubuntu/jenkins-pipeline/leafhub
                        
                        git checkout $COMMIT_ID 
                        
                        nohup sudo mvn spring-boot:run > /dev/null 2>&1 & 
                   
                   '
                   '''
               }
               
               
               
            }
            
            
            
        }
 	stage('Sanity API Tests on QA') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                // This will pause the pipeline for 30 seconds for the dev server to be ready
                sleep(10)
                
                sshagent(['dev-ssh-credentials']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@${TEST_SERVER_IP} '
                    
                    # Update the package lists for upgrades and new package installations
                    sudo apt-get update

                    # Install OpenJDK 8
                    sudo apt-get install -y openjdk-8-jdk

                    # Install Maven
                    sudo apt-get install -y maven

                    # Clone the webdriver-tests repository
                    git clone https://github.com/TestLeafInc/api-leafhub-tests

                    # Move to the webdriver-tests folder
                    cd api-leafhub-tests

                    # pull the changes if any
                    git pull
                    
                    # Create report directory
                    mkdir reports
                    
                    # Run Maven tests
                    mvn clean test

                    # Push the results to S3 (make sure to install and configure awsconfigure before)
                    aws s3 sync reports/ s3://reports-html-api
                       
                    '
                    '''
                }
            }
        }
        
    stage('Sanity Web Tests on QA') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                // This will pause the pipeline for 30 seconds for the dev server to be ready
                sleep(10)
                
                sshagent(['dev-ssh-credentials']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@${TEST_SERVER_IP} '
                    
                    # Update the package lists for upgrades and new package installations
                    sudo apt-get update

                    # Install OpenJDK 8
                    sudo apt-get install -y openjdk-8-jdk

                    # Install Maven
                    sudo apt-get install -y maven

                    # Install xvfb
                    sudo apt-get install -y xvfb

                    # Start xvfb
                    Xvfb :99 &
                    export DISPLAY=:99

                    # Download and install Google Chrome
                    if ! command -v google-chrome > /dev/null; then
                    wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
                    sudo dpkg -i google-chrome-stable_current_amd64.deb
                    sudo apt --fix-broken install -y
                    else
                    echo "Google Chrome is already installed."
                    fi


                    # Clone the webdriver-tests repository
                    git clone https://github.com/TestLeafInc/webdriver-leafhub

                    # Move to the webdriver-tests folder
                    cd webdriver-leafhub

                    # pull the changes if any
                    git pull
                    
                    # Run Maven tests
                    mvn clean test -DsuiteXmlFile=sanity.xml -Dserver.ip=65.2.132.29

                    # Push the results to S3 (make sure to install and configure awsconfigure before)
                    aws s3 sync reports/ s3://reports-html-selenium
                       
                    '
                    '''
                }
            }
        }

    }
}
