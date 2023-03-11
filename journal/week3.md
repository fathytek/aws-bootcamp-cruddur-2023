# Week 3 — Decentralized Authentication

## Authentication to AWS 

Through your shell (bash) Add Autocompletion 

 ```sh
find / -name aws_completer
echo $SHELL
ls -a ~/
export PATH=/usr/local/bin/:$PATH
source ~/.bash_profile
 complete -C '/usr/local/bin/aws_completer' aws
 ```
Check your are connect to you AWS account using aws cli command

 ```
 aws sts get-caller-identity
 ```

 ## Install AWS Amplify

 AWS Amplify is a complete solution that lets frontend web and mobile developers easily build, ship, and host full-stack applications on AWS, with the flexibility to leverage the breadth of AWS services as use cases evolve. No cloud expertise needed.



```sh
npm i aws-amplify --save
```
