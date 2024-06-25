pipeline {
    agent any
    
    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
    }
    
    stages {
        stage('Creating and Starting EC2 Instance') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'riya_kochar'
                ]]) {
                    script {
                        // Creating EC2 Instance
                        def instanceId = bat(script: '''
                            aws ec2 run-instances --image-id ami-04b70fa74e45c3917 --count 1 --instance-type t2.micro --key-name riya-kochar --subnet-id subnet-05c8a47e3f7a5c54f --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=riya-machine}]"
                            aws ec2 describe-instances --filters "Name=tag:Name,Values=riya-machine" --query "Reservations[].Instances[].InstanceId" --output text
                        ''', returnStdout: true).split('\n')[-1].trim()
                        
                        bat(script: "aws ec2 wait instance-running --instance-ids ${instanceId}")
                        
                        env.INSTANCE_ID = instanceId
                    }
                }
            }
        }
        stage('Allocating Elastic IP') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'riya_kochar'
                ]]) {
                    script {
                        def allocateIpCmd = bat(script: "aws ec2 allocate-address --domain vpc --network-border-group us-east-1", returnStdout: true).trim()
                        
                        def allocationId = allocateIpCmd =~ /"AllocationId":\s*"([^"]+)"/
                        def elasticIp = allocateIpCmd =~ /"PublicIp":\s*"([^"]+)"/
                        
                        if (!allocationId || !elasticIp) {
                            error "Failed to allocate Elastic IP"
                        }
                        
                        env.ALLOCATION_ID = allocationId[0][1]
                        env.ELASTIC_IP = elasticIp[0][1]
                        echo "Allocation ID: ${env.ALLOCATION_ID}"
                        echo "Elastic IP: ${env.ELASTIC_IP}"
                    }
                }
            }
        }
        stage('Associate Elastic IP') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'riya_kochar'
                ]]) {
                    script {
                        bat(script: "aws ec2 associate-address --instance-id ${env.INSTANCE_ID} --public-ip ${env.ELASTIC_IP}")
                    }
                }
            }
        }
    }
}