
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
		
        stage('Tests') {
            parallel {
                 stage('Unit') {
                    agent { label 'agent1'}
                    steps {
                        script {
                        def stageName = env.STAGE_NAME
                        echo "Stage: ${stageName}"
                        def nodeName = env.NODE_NAME
                        echo "Agent: ${nodeName}"
                        }
                         unstash name:'code'
                         catchError(buildResult:'UNSTABLE',stageResult:'FAILURE') {
                             bat ''' 
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-unit.xml test\\unit
                            '''
                           junit testResults: 'result-unit.xml'
                           }
                        }
                }
                 stage('Rest') {
                    agent { label 'agent2'}
                    steps {
                        script {
                        def stageName = env.STAGE_NAME
                        echo "Stage: ${stageName}"
                        def nodeName = env.NODE_NAME
                        echo "Agent: ${nodeName}"
                        }
                        unstash name:'code'
                        catchError(buildResult:'UNSTABLE',stageResult:'FAILURE') {
                            bat '''    
                            set FLASK_APP=app\\api.py
                            start /B flask run
                            
                            start /wait timeout 7
                            
                            set PYTHONPATH=.
                            start java -jar C:\\Users\\adan.garciagarcia\\Desktop\\CursoDevops\\Herramientas\\wiremock-standalone-3.5.4.jar --port 9090 --verbose --root-dir test\\wiremock
                            pytest --junitxml=result-rest.xml test\\rest
                        '''
                        junit testResults:'result-rest.xml'
                        }
                    }
                }
                stage('Performance'){
                agent {label 'agent3'}
                steps {
                    script {
                        def stageName = env.STAGE_NAME
                        echo "Stage: ${stageName}"
                        def nodeName = env.NODE_NAME
                        echo "Agent: ${nodeName}"
                    }
                    unstash name:'code'
                    catchError(buildResult:'UNSTABLE',stageResult:'FAILURE') {
                    bat 'C:\\Users\\adan.garciagarcia\\Desktop\\CursoDevops\\Herramientas\\apache-jmeter-5.6.3\\apache-jmeter-5.6.3\\bin\\jmeter -n -t test\\jmeter\\flask.jmx -f -l flask.jtl'
                    perfReport sourceDataFiles : 'flask.jtl'
                      }
                }
                }
            }
        }
        stage('Static') {
			steps{
               
                bat '''
                    flake8 --format=pylint --extend-ignore E501 --exit-zero  app >flake8.out
                '''
                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 8, type: 'TOTAL', unstable: true],[threshold:10, type: 'TOTAL', unhealthy: true]]
                }
			}
		}
        stage('Cobertura'){
            steps {
                    bat '''
                        coverage run --branch --source=app --omit=app\\__init__.py,app\\api.py -m pytest test\\unit
                        coverage xml
                    '''
                    catchError(buildResult:'FAILURE',stageResult:'FAILURE') {
                        cobertura coberturaReportFile:'coverage.xml', onlyStable:false, failUnstable:false, conditionalCoverageTargets:'100,80,90',  lineCoverageTargets:'100,85,95'
                   }
            }
        }
		stage('Security') {
			steps {
            catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
				bat '''
					bandit -r .  --exit-zero -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}"
				'''
				recordIssues tools: [pyLint(name: 'bandit', pattern: 'bandit.out')], qualityGates: [[threshold:2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unhealthy: true]]
                }
            }
		}
    }
}
