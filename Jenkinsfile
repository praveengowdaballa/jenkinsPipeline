pipeline {
    agent any

    parameters {
        string(name: 'ACCOUNT_ID', description: 'AWS Account ID')
        string(name: 'REGION', description: 'AWS Region')
		string(name: 'AMIID', description: 'AWS AMIID')
    }

    stages {
        stage('Assume Role and Fetch VPCs') {
            steps {
                script {
                    def MRoleARN = "arn:aws:iam::123456789:role/DevOps-Automation"
                    def ARoleARN = "arn:aws:iam::${params.ACCOUNT_ID}:role/Cloud-Governance-Parent-Account-Access-Role"
                    
                    def Auth = "stsexec --rolearn ${MRoleARN} --rolearn ${ARoleARN}"
                    def awsCliCommand = "${Auth} aws --region ${params.REGION} ec2 describe-vpcs --query 'Vpcs[*].VpcId' | tr -d '[],\"'"
                    env.VPC_OPTIONS = sh(script: awsCliCommand, returnStdout: true).trim()
                }
            }
        }

        stage('Input VPC') {
            steps {
                script {
                    def vpcChoices = env.VPC_OPTIONS.split("\n").collect { it.trim() }
                    def userInput = input(
                        id: 'VPC_Input',
                        message: 'Select VPC:',
                        parameters: [choice(name: 'VPC_ID', choices: vpcChoices)]
                    )
                    env.VPC_ID = userInput
                }
            }
        }

        stage('Fetch Subnets for Selected VPC') {
            steps {
                script {
                    def MRoleARN = "arn:aws:iam::123456789:role/DevOps-Automation"
                    def ARoleARN = "arn:aws:iam::${params.ACCOUNT_ID}:role/Cloud-Governance-Parent-Account-Access-Role"
                    
                    def Auth = "stsexec --rolearn ${MRoleARN} --rolearn ${ARoleARN}"
                    def awsCliCommand = "${Auth} aws --region ${params.REGION} ec2 describe-subnets --filters Name=vpc-id,Values=${env.VPC_ID} --query 'Subnets[*].SubnetId' | tr -d '[],\"'"
                    env.SUBNET_OPTIONS = sh(script: awsCliCommand, returnStdout: true).trim()
                }
            }
        }

        stage('Input Subnet') {
            steps {
                script {
                    def subnetChoices = env.SUBNET_OPTIONS.split("\n").collect { it.trim() }
                    def userInput = input(
                        id: 'Subnet_Input',
                        message: 'Select Subnet:',
                        parameters: [choice(name: 'SUBNET_ID', choices: subnetChoices, description: 'Select one subnet')]
                    )
                    env.SUBNET_ID = userInput
                    echo "Selected Subnet ID: ${env.SUBNET_ID}"
                    
                }
            }
        }

        stage('Deploy Instance') {
            steps {
                script {
                    def MRoleARN = "arn:aws:iam::123456789:role/DevOps-Automation"
                    def ARoleARN = "arn:aws:iam::${params.ACCOUNT_ID}:role/Cloud-Governance-Parent-Account-Access-Role"
                    
                    def Auth = "stsexec --rolearn ${MRoleARN} --rolearn ${ARoleARN}"
                    def awsCliCommand = "${Auth} aws --region ${params.REGION} ec2 run-instances --image-id ${params.AMIID} --instance-type t2.micro --subnet-id ${env.SUBNET_ID} --security-group-ids sg-0acb3e10f2cd85f5f --key-name ccoe-demo-12042024"
                    sh(awsCliCommand)
                }
            }
        }

        // Add more stages as needed
    }
}
