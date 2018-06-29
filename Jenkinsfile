pipeline {
	agent {
		node { label 'Linux' // allocate a node/slave ex: '' or '' or 'master'
			}
		}				
	stages {				
		stage('Build') {
            steps {	// Secify nodejs version
						nodejs(nodeJSInstallationName:'NodeJS6.9.1') {
						sh ' mkdir ${WORKSPACE}/dlnpm '	
						sh ' npm config set tmp ${WORKSPACE}/dlnpm '
						sh ' chmod 755 jenkins/prstatus/*.sh'
						sh './jenkins/prstatus/eslintprps.sh'
						sh './jenkins/prstatus/coverageprps.sh'
						sh 'npm install' //Install npm packages and Run grunt build
						// stash includes: './**/*', name: 'workspacesources'
						}
					}
				}
		stage('Linting') {
            steps {	// Specify nodejs version
						nodejs(nodeJSInstallationName:'NodeJS6.9.1') {
						//Run Static Code Analysis using Eslint, htmllint,Csslint
						// Generating ESLINT HTML Report   
						sh './node_modules/.bin/eslint -f html ./app/assets/js/**/* >eslint.html || echo "Linting failed/Success, continuing..." '
						// Generating CheckStyle Reports
						sh './node_modules/.bin/eslint -f checkstyle ./app/assets/js/**/* >eslint.xml || echo "Linting failed/Success, continuing..." '
						// ESLINT, CSS, HTML Lint 
						sh 'npm run lint || echo " continuing to skip errors..." '
						//sh './node_modules/.bin/eslint -f junit ./app/assets/js/**/* >junit.xml || echo "Linting failed/Success, continuing..." '
						// stash includes: './**/*', name: 'workspacesources'
						publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '', reportFiles: 'eslint.html', reportName: 'ESLint Report', reportTitles: ''])
						checkstyle canComputeNew: false, defaultEncoding: '', healthy: '', pattern: 'eslint.xml', unHealthy: ''
						sh './jenkins/prstatus/eslintprus.sh'
						}
					}
				}
		stage('CodeCoverage') {
            steps {	// Specify nodejs version
						nodejs(nodeJSInstallationName:'NodeJS6.9.1') {
						// Run Istanbul code coverage 
						sh ' npm run cover || :'
						publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'coverage/lcov-report/', reportFiles: 'index.html', reportName: 'Istanbul Coverage Report', reportTitles: 'Code Coverage'])
						sh './jenkins/prstatus/coverageprus.sh'
						}
					}
				}
		stage('SonarQube Analysis') {
			steps {
			      withSonarQubeEnv('SonarQube6.5-Linux') {
				      sh ' cd /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQube_Scanner_3.0.3.778/bin ; export SONAR_SCANNER_OPTS="-Xmx3062m -XX:MaxPermSize=512m -XX:ReservedCodeCacheSize=128m" ; ./sonar-scanner ' + 
						'-Dsonar.sourceEncoding=UTF-8 ' + 
						'-Dsonar.eslint.eslintconfigpath=${WORKSPACE}/.eslintrc.js ' +  
						'-Dsonar.javascript.lcov.reportPaths=${WORKSPACE}/coverage/lcov.info ' + 
						'-Dsonar.sources=. ' + 
						'-Dsonar.language=js ' + 
						'-Dsonar.projectVersion=0.0.1.${BUILD_ID} ' + 
						'-Dsonar.projectKey=UO_WEB_ICE ' +    
						'-Dsonar.projectName=UO_WEB_ICE ' +
						'-Dsonar.exclusions=node_modules/**/*,app/dist/**/*,protractor/**/*,dlnpm/**/*,app/bower_components/**/*,app/assets/js/app/uoAppWebStore/**/*,app/assets/js/app/buildingblocks/directives/toggleHandleB.directive.js,app/assets/js/app/uoApp/controllers/upApp.footer.controller.js,app/assets/js/app/uoApp/controllers/upApp.header.controller.js,app/assets/js/app/uoApp/controllers/upApp.mainContent.controller.js,app/assets/js/bare/bootstrap/bootstrap.module.js,app/assets/js/bare/bootstrap/config/UOAppBootstrap.run.js,app/assets/js/bare/bootstrap/config/bootstrap.constants.js,app/assets/js/bare/bootstrap/config/http.config.js,app/assets/js/bare/bootstrap/factories/k2UrlInformation.factory.js,app/assets/js/bare/bootstrap/factories/localStorageLarge.factory.js ' +
						'-Dsonar.projectBaseDir=${WORKSPACE} '
					}
				}
			post {      
                success {
	                step([$class: 'GitHubCommitStatusSetter',
		                    contextSource: [$class: 'ManuallyEnteredCommitContextSource', context: 'SonarQube Analysis'],
							statusBackrefSource: [$class: 'ManuallyEnteredBackrefSource', backref: 'http://SONAR.net:9000/dashboard?id=UO_WEB_ICE'],
		                    statusResultSource: [$class: 'ConditionalStatusResultSource', 
		                    results: [[$class: 'AnyBuildResult', message: 'Yay ! Successful', result: 'SUCCESS', state: 'SUCCESS']]]
		                    ])
	                    }
                failure {
	                step([$class: 'GitHubCommitStatusSetter',
		                    contextSource: [$class: 'ManuallyEnteredCommitContextSource', context: 'SonarQube Analysis'],
							statusBackrefSource: [$class: 'ManuallyEnteredBackrefSource', backref: 'http://sonar01.net:9000/dashboard'],
		                    statusResultSource: [$class: 'ConditionalStatusResultSource', 
		                    results: [[$class: 'AnyBuildResult', message: 'Failed', result: 'UNSTABLE', state: 'FAILURE']]]
		                    ])
	                    }	                    
                    }				
			}
		stage('Quality Gate'){
		      steps { 
			  script {
			      sleep 60
		          timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
                    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
					echo ""
                    if (qg.status != "OK") {
                     error "Pipeline aborted due to quality gate failure: ${qg.status}"
							}
						}
					}
				}
			post {      
                success {
	                step([$class: 'GitHubCommitStatusSetter',
		                    contextSource: [$class: 'ManuallyEnteredCommitContextSource', context: 'SonarQube Quality Gate Check'], 
							statusBackrefSource: [$class: 'ManuallyEnteredBackrefSource', backref: 'http://sonar01:9000/dashboard'],
		                    statusResultSource: [$class: 'ConditionalStatusResultSource', 
		                    results: [[$class: 'AnyBuildResult', message: 'Yay ! OK', result: 'SUCCESS',  state: 'SUCCESS']]]
		                    ])
	                    }
                failure {
	                step([$class: 'GitHubCommitStatusSetter',
		                    contextSource: [$class: 'ManuallyEnteredCommitContextSource', context: 'SonarQube Quality Gate Check'],
							statusBackrefSource: [$class: 'ManuallyEnteredBackrefSource', backref: 'http://sonar01:9000/dashboard'],
		                    statusResultSource: [$class: 'ConditionalStatusResultSource', 
		                    results: [[$class: 'AnyBuildResult', message: 'Oh no its not OK ', result: 'UNSTABLE', state: 'FAILURE']]]
		                    ])
	                    }	                    
                    }	
			}			
			
		}
		
	post {

		always{ 					
		deleteDir()	
		}
		       // success {
              // step([$class: 'CompareCoverageAction'])
        //}
		failure {	
			emailext (
				subject: "FAILED  Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
				body: """<p>FAILED  Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' : </p>
					<p>Some one broke the build. Please Check the build log attached for errors </p>""",
				attachLog: true, from: 'e-mail@.com', replyTo: 'donotreply@.com', to: 'e-mail@.com'
				)
			}
		}	

	// The options directive is for configuration that applies to the whole job.
	options {
			// For example, we'd like to make sure we only keep 10 builds at a time, so
			// we don't fill up our storage!
		buildDiscarder(logRotator(numToKeepStr:'10'))
    
			// And we'd really like to be sure that this build doesn't hang forever, so
			// let's time it out after an hour.
		timeout(time: 60, unit: 'MINUTES')
		}
		
	}
