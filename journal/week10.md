# Week 10 — CloudFormation Part 1

gitpod /workspace/aws-bootcamp-cruddur-2023 (main) $ cfn-guard rulegen --template /workspace/aws-bootcamp-cruddur-2023/aws/cfn/template.yaml 
let aws_ecs_cluster_resources = Resources.*[ Type == 'AWS::ECS::Cluster' ]
rule aws_ecs_cluster when %aws_ecs_cluster_resources !empty {
  %aws_ecs_cluster_resources.Properties.CapacityProviders == ["FARGATE"]
  %aws_ecs_cluster_resources.Properties.ClusterName == "MyCluster"
}

![image](https://user-images.githubusercontent.com/32872009/235468579-2e7d71fd-2309-462f-af23-a0a1e0a2089e.png)