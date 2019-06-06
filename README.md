# Spinnaker Installation

# Method 1: Use Spinnaker AMI  without cloud formation on AWS

  Step-1 : Create EC2 instance from AMI

  Below are the region wise AMI ids which we can use: 
  ```
  "SpinnakerAMIId": {
            "sa-east-1": {
                "AMI": "ami-f3e1769f"
            },
            "us-east-1": {
                "AMI": "ami-4153d956"
            },
            "us-west-1": {
                "AMI": "ami-87dc9de7"
            },
            "us-west-2": {
                "AMI": "ami-21a26d41"
            },
            "eu-central-1": {
                "AMI": "ami-0c88d786f0a9d2d53"
            },
            "eu-west-1": {
                "AMI": "ami-096219de2c93ff4fb"
            },
            "eu-west-2": {
                "AMI": "ami-0b0c46c863dce9c9e"
            },
            "ap-northeast-2": {
                "AMI": "ami-018e5a7c42f44d29d"
            },
            "ap-southeast-2": {
                "AMI": "ami-0d0e906aca361b071"
            },
            "ap-southeast-1": {
                "AMI": "ami-004af9ef7fb13265a"
            },
            "ca-central-1": {
                "AMI": "ami-060ee9b36057db545"
            }
        }
 ```
        
  Create EC2 instance from anyone of the above AMI's.

# Step-2 : Create BaseAMI role using cloud formation or manually

  Use below clodformation template for creation of role:
 ```
 {
    "AWSTemplateFormatVersion": "2010-09-09",
    "Description": "CreateBaseRole",
    "Resources": {
        "BaseIAMRole": {
            "Properties": {
                "AssumeRolePolicyDocument": {
                    "Statement": [
                        {
                            "Action": [
                                "sts:AssumeRole"
                            ],
                            "Effect": "Allow",
                            "Principal": {
                                "Service": [
                                    "ec2.amazonaws.com"
                                ]
                            }
                        }
                    ],
                    "Version": "2012-10-17"
                },	
                "Path": "/"
            },
            "Type": "AWS::IAM::Role"
        }
    },
    "Outputs": {
    }
}
```
# Step-3 : Update AWS Provider configuration and restart spinnaker

    /opt/spinnaker/bin/stop_spinnaker.sh
    /var/lib/dpkg/info/ca-certificates-java.postinst configure
    sed -i 's/name: .*/name: default/g' /opt/spinnaker/config/spinnaker-local.yml
    sed -i 's/defaultIAMRole: .*/defaultIAMRole: <<<<REPLACE WITH BASEAMIRoleName>>>>/g' /opt/spinnaker/config/spinnaker-local.yml
    sed -i 's/SPINNAKER_AWS_ENABLED=false/SPINNAKER_AWS_ENABLED=true/g' /etc/default/spinnaker
    sed -i 's/SPINNAKER_AWS_DEFAULT_REGION=.*/SPINNAKER_AWS_DEFAULT_REGION=us-west-1/g' /etc/default/spinnaker
    mkdir /home/spinnaker/.aws
    printf "[default]\naws_access_key_id=<<AWS_ACCESS_KEY>> \naws_secret_access_key=<<AWS_SECRET_KEY>>" > /home/spinnaker/.aws/credentials
    mkdir /root/.aws
    printf "[default]\naws_access_key_id=<<AWS_ACCESS_KEY>>  \naws_secret_access_key=<<AWS_SECRET_KEY>>" > /root/.aws/credentials

    /opt/spinnaker/scripts/reconfigure_spinnaker.sh
    /opt/spinnaker/bin/start_spinnaker.sh


# Step-4 : Create SSH tunnel to connect to spinnaker and access the spinnaker UI

For windows create below file :

c:/Users/<<USER>>/.ssh/config:

```
Host spinnaker
    HostName <<REPLACE WITH PUBLIC DNS OF EC2 INSTANCE>>
    IdentityFile /path/to/private/key
    LocalForward 9000 127.0.0.1:9000
    LocalForward 8084 127.0.0.1:8084
    LocalForward 8087 127.0.0.1:8087
    User ubuntu
```

  Execute: ssh -f -N spinnaker
  Open http://localhost:9000/ in your web browser
 
 
 References:
 https://www.gremlin.com/chaos-monkey/advanced-developer-guide/
 https://www.spinnaker.io/setup/install/deploy/
 https://www.spinnaker.io/setup/quickstart/faq//#i-want-to-expose-localdebian-spinnaker-on-a-public-ip-address-but-it-always-binds-to-localhost
 https://www.spinnaker.io/guides/tutorials/codelabs/hello-deployment/
 
