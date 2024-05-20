
pipeline {
    agent any
    options {
        parallelsAlwaysFailFast()
        }
     stages {
        stage('Get Code') {
            steps {
                git branch: 'master', url:'https://github.com/Adan2GG/unir-CP1-B.git'
                stash name:'code' , includes:'**'
            }
        }
		
		stage('Static') {
			steps{
				bat '''
					flake8 --format=pylint --exit-zero --extend-ignore E271,E501 app >flake8.out
				'''
				recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold:10, type: 'TOTAL', unstable: false], [threshold: 5, type: 'TOTAL', unstable: false]]
			}
		}
        stage('Tests') {
            parallel {
                 stage('Unit') {
                    agent { label 'agent1'}
                    steps {
                         unstash name:'code'
                         catchError(buildResult:'UNSTABLE',stageResult:'FAILURE') {
                             bat ''' 
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-unit.xml test\\unit
                            '''
                            stash name:'unit-res', includes:'result-unit.xml'
                           }
                        }
                }
                 stage('Rest') {
                    agent { label 'agent2'}
                    steps {
                        unstash name:'code'
                        catchError(buildResult:'UNSTABLE',stageResult:'FAILURE') {
                            bat '''    
                            set FLASK_APP=app\\api.py
                            start flask run
                            
                            start /wait timeout 10
                            
                            set PYTHONPATH=.
                            start java -jar C:\\Users\\adan.garciagarcia\\Desktop\\CursoDevops\\Herramientas\\wiremock-standalone-3.5.4.jar --port 9090 --verbose --root-dir test\\wiremock
                            pytest --junitxml=result-rest.xml test\\rest
                        '''
                            stash name:'rest-res', includes:'result-rest.xml'
                        }
                    }
                }
            }
        }
        stage('cobertura'){
            steps {
                unstash name:'code'
                bat '''
                    coverage run --branch --source=app --omit=app\\__init__.py,app\\api.py -m pytest test\\unit
                    coverage xml
                '''
                    catchError(buildResult:'UNSTABLE',stageResult:'FAILURE') {
                        cobertura coberturaReportFile:'coverage.xml', onlyStable:false, failUnstable:false, conditionalCoverageTargets:'10,10,10',  lineCoverageTargets:'100,80,98'
                    }
            }
            post {
                always {
                    // Define el color del estado de la etapa
                    script {
                        currentBuild.result = 'SUCCESS' // Por defecto, establecemos el estado a SUCCESS
                        if (currentBuild.result == 'UNSTABLE') {
                            currentBuild.result = 'UNSTABLE'
                        }
                    }
                }
            }
        }
		stage('security') {
			steps {
				bat '''
					bandit -r .  --exit-zero -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}"
				'''
				recordIssues tools: [pyLint(name: 'bandit', pattern: 'bandit.out')], qualityGates: [[threshold:1, type: 'TOTAL', unstable: true], [threshold: 3, type: 'TOTAL', unstable: false]]
			}
		}
       
        stage('Result') {
            steps {
                unstash name:'unit-res'
                unstash name:'rest-res'
                junit 'result*.xml'
            }
        }
		stage('clear') {
			 steps {
                bat '''
                    REM Limpia el workspace
                    git clean -fd

                    REM Limpia el stash
                    git stash clear
                '''
            }
		}
    }
}
