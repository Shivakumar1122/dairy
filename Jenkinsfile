pipeline{
  agent {label 'main'}
    stages{
       stage('hosting application'){
        steps{
          sh "ls"
          sh "aws rds create-db-instance --db-instance-identifier test-mysql-instance2 --db-name dfsms --db-instance-class db.t2.micro --vpc-security-group-ids  sg-04638412bef8421f5 --engine mysql --engine-version 5.7 --db-parameter-group-name default.mysql5.7 --publicly-accessible  --master-username admin --master-user-password shiva123 --allocated-storage 10 --region us-east-2"
          sleep(450)
          script{
              def cmd = "aws rds describe-db-instances --db-instance-identifier test-mysql-instance2 --region us-east-2"
              def output = sh(script: cmd,returnStdout: true)
              jsonitem = readJSON text: output
              println(jsonitem)
           }
           sh "sed -i.bak 's/endpoint/${jsonitem['DBInstances'][0]['Endpoint']['Address']}/g' userdata.txt"
          script{
              def cmd = "aws elbv2 create-load-balancer --name my-load-balancer --subnets subnet-0e7d0fc5926bcbb6d subnet-0fbbde030ff886308 --security-groups sg-04638412bef8421f5 --type network --region us-east-2 "
              def output = sh(script: cmd,returnStdout: true)
              jsonitem1 = readJSON text: output
              println(jsonitem1)
              sleep(100)
            }
          script{
              def cmd = "aws elbv2 create-target-group --name my-targets --protocol TCP --port 80 --target-type instance --vpc-id vpc-021ac10c8a336a064 --region us-east-2"
              def output = sh(script: cmd,returnStdout: true)
              jsonitem2 = readJSON text: output
              println(jsonitem2)
              sleep(180)
               }
           sh "aws elbv2 create-listener --load-balancer-arn ${jsonitem1['LoadBalancers'][0]['LoadBalancerArn']} --protocol TCP --port 80 --default-actions Type=forward,TargetGroupArn=${jsonitem2['TargetGroups'][0]['TargetGroupArn']} --region us-east-2"
           sh "aws autoscaling create-launch-configuration --launch-configuration-name my-lc3-cli --image-id ami-0fb653ca2d3203ac1 --instance-type t2.micro --security-groups sg-04638412bef8421f5 --key-name shivakey --iam-instance-profile demorole --user-data file://userdata.txt --region us-east-2"
           sh "aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg1-cli --launch-configuration-name my-lc3-cli --max-size 2 --min-size 1 --desired-capacity 1 --target-group-arns ${jsonitem2['TargetGroups'][0]['TargetGroupArn']} --availability-zones us-east-2c --region us-east-2"
           sh "aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg2-cli --launch-configuration-name my-lc3-cli --max-size 2 --min-size 1 --desired-capacity 1 --target-group-arns ${jsonitem2['TargetGroups'][0]['TargetGroupArn']} --availability-zones us-east-2c --region us-east-2"
           sh "aws autoscaling put-scaling-policy --auto-scaling-group-name my-asg1 --policy-name alb1000-target-tracking-scaling-policy --policy-type TargetTrackingScaling --target-tracking-configuration file://config.json --region us-east-2"
           sh "aws autoscaling put-scaling-policy --auto-scaling-group-name my-asg2 --policy-name alb1000-target-tracking-scaling-policy --policy-type TargetTrackingScaling --target-tracking-configuration file://config.json --region us-east-2"
        }
       }
  }
}
