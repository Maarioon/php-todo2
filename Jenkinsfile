pipeline {
    agent any
    
    stages {
        stage('Initial Cleanup') {
            steps {
                dir("${WORKSPACE}") {
                    // deleteDir()
                }
            }
        }
        
        stage('Checkout SCM') {
            steps {
                script {
                    echo "Checking out source code from SCM..."
                    // git branch: 'main', 
                    //     credentialsId: 'your-credentials-id', 
                    //     url: 'https://github.com/Maarioon/php-todo2-jk.git'
                }
            }
        }
        
        stage('Prepare Dependencies') {
            steps {
                script {
                    echo "Preparing dependencies..."
                    // sh 'mv .env.sample .env'
                    // sh 'composer install'
                    // sh 'php artisan migrate'
                    // sh 'php artisan db:seed'
                    // sh 'php artisan key:generate'
                }
            }
        }
        
        stage('Execute Unit Tests') {
            steps {
                script {
                    echo "Executing unit tests..."
                    // sh './vendor/bin/phpunit'
                }
            }
        }
        
        stage('Code Analysis') {
            steps {
                script {
                    echo "Performing code analysis..."
                    sh 'mkdir -p build/logs'
                    sh 'phploc app/ --log-csv build/logs/phploc.csv'
                }
            }
        }
        
        stage('Plot Code Coverage Report') {
            steps {
                script {
                    plot csvFileName: 'plot-code-coverage.csv',
                         csvSeries: [[
                             file: 'build/logs/phploc.csv',
                             inclusionFlag: 'INCLUDE_BY_STRING'
                         ]],
                         group: 'phploc',
                         numBuilds: '100',
                         style: 'line',
                         title: 'Code Coverage Metrics',
                         yaxis: 'Metrics'
                }
            }
        }
        
        stage('Package Artifact') {
            steps {
                script {
                    echo "Packaging application as artifact..."
                    sh 'zip -qr php-todo-${BUILD_ID}.zip ${WORKSPACE}/*'
                }
            }
        }
        
        stage('Upload Artifact to Artifactory') {
            steps {
                script {
                    echo "Uploading artifact to Artifactory..."
                    def server = Artifactory.server('artifactory')
                    def uploadSpec = """{
                        "files": [{
                            "pattern": "php-todo-${BUILD_ID}.zip",
                            "target": "todo-dev-local-generic-local/php-todo-${BUILD_ID}.zip",
                            "props": "type=zip;status=ready"
                        }]
                    }"""
                    server.upload spec: uploadSpec
                }
            }
        }
        
        stage('Deploy to Dev Environment') {
            steps {
                build job: 'ansible-config-mgt/main',
                      parameters: [[
                          $class: 'StringParameterValue',
                          name: 'inventory',
                          value: 'dev'
                      ]],
                      propagate: false,
                      wait: true
            }
        }
        
        stage('SonarQube Quality Gate') {
            when {
                branch pattern: "^develop*|^hotfix*|^release*|^main*",
                comparator: "REGEXP"
            }
            environment {
                scannerHome = tool 'SonarQubeScanner'
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "${scannerHome}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
                }
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
