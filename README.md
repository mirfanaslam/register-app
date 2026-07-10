register-app
<br>
Test93

---------
### in short form line by line
stage("Trivy Scan") {
    steps {
        script {
            sh '''
            docker run \
              -v /var/run/docker.sock:/var/run/docker.sock \
              aquasec/trivy image \
              ashfaque9x/register-app-pipeline:latest \
              --no-progress \
              --scanners vuln \
              --exit-code 0 \
              --severity HIGH,CRITICAL \
              --format table
            '''
        }
    }
}
### OR in long form.
stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ashfaque9x/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }


