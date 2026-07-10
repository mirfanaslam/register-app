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

### difference
| Feature        | First Stage                | Second Stage                            |
| -------------- | -------------------------- | --------------------------------------- |
| How Trivy runs | Inside a Docker container  | Installed directly on the Jenkins agent |
| Output         | Printed to Jenkins console | Saved to report files                   |
| Report format  | Table in console           | JSON and text files                     |
| Dependency     | Requires Docker            | Requires Trivy installed on the agent   |
| Best for       | Quick vulnerability check  | CI/CD reporting and archiving           |

### same trivy in an other way
stage("Trivy Scan Image") {
	steps {
		script {
			sh """ echo ' Running Trivy scan on ${env.IMAGE_TAG}' 
			# JSON report 
			trivy image -f json -o trivy-image.json ${env.IMAGE_TAG} 
			# HTML report using built-in HTML format 
			trivy image -f table -o trivy-image.txt ${env.IMAGE_TAG} 
			# Fail build if HIGH/CRITICAL vulnerabilities found  
			trivy image --exit-code 1 --severity HIGH,CRITICAL ${env.IMAGE_TAG} || true """ } } }

