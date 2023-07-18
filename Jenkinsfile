node {
    stage('Build') {
        node {
            docker.image('python:2-alpine').inside {
                  sh '''
        echo "Current Directory: $(pwd)"
        echo "Files in Directory:"
        ls -la
        '''
                sh 'python -m py_compile /sources/add2vals.py /sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
    }

    stage('Test') {
        node {
            docker.image('qnib/pytest').inside {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
        }
        post {
            always {
                junit 'test-reports/results.xml'
            }
        }
    }

    stage('Manual Approve') {
        node {
            input message: 'Lanjutkan ke tahap Deploy? (Klik "Proceed" untuk lanjut)'
        }
    }

    stage('Deploy') {
        node {
            env.VOLUME = "${pwd()}/sources:/src"
            env.IMAGE = 'cdrx/pyinstaller-linux:python2'

            unstash(name: 'compiled-results')
            sh "docker run --rm -v ${env.VOLUME} ${env.IMAGE} 'pyinstaller -F add2vals.py'"
            sh 'sleep 1m'
        }
        post {
            success {
                archiveArtifacts 'sources/dist/add2vals'
                sh "docker run --rm -v ${env.VOLUME} ${env.IMAGE} 'rm -rf build dist'"
            }
        }
    }
}
