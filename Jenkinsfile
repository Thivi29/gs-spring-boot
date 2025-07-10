pipeline {
    agent any
    stages {
        stage('build') {
            steps {
                sh '''
                cd initial
                ./mvnw package
                '''
            }
        }

        stage('deploy') {
            steps {
                sh '''
                USER=petclinicapp
                GROUP=petclinicapp
                S3_BUCKET=petclinicapp-10-07
                BUILD_FILE_NAME=petclinicapp-v1.jar
                LOCAL_FILE_PATH=/home/$USER/$BUILD_FILE_NAME

                # Rename JAR
                cp initial/target/spring-boot-initial-0.0.1-SNAPSHOT.jar initial/target/$BUILD_FILE_NAME

                # Upload to S3
                aws s3 cp $WORKSPACE/initial/target/$BUILD_FILE_NAME s3://$S3_BUCKET

                # Get running EC2 instance IPs
                output=$(aws ec2 describe-instances \
                    --filter "Name=tag:appname,Values=petclinic" "Name=instance-state-name,Values=running" \
                    --query "Reservations[*].Instances[*].[InstanceId,PublicIpAddress]" \
                    --output json)

                ips=$(echo "$output" | jq -r '.[][][1]')
                echo "$ips"

                for ip in $ips
                do
                    echo "Connecting to $ip"
                    ssh -o StrictHostKeyChecking=no -i /home/jenkins/petclinicappkey.pem ubuntu@$ip bash << 'EOSSH'
                        sudo aws s3 cp s3://petclinicapp-10-07/petclinicapp-v1.jar /home/petclinicapp/petclinicapp-v1.jar
                        sudo systemctl restart petclinicapp.service
                        echo "Updated and restarted on $ip"
EOSSH
                done
                '''
            }
        }
    }
}
