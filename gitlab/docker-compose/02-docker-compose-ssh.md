# docker compose project 

## Evolutions-Phase 2: Anwenden eines Docker Compose files über ssh 

### Schritt 1: gitlab-ci.yaml 

```


workflow:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "web"'

default:
  image: alpine
stages:          # List of stages for jobs, and their order of execution
  - deploy 

deploy-job:
   stage: deploy   
   image: ubuntu 
  
   before_script:
    - apt-get -y update
    - apt-get install -y openssh-client 
    - apt-get install -y ca-certificates curl gnupg lsb-release
    - mkdir -p /etc/apt/keyrings
    # We want the newest version from docker
    # version from ubuntu repo does not work (docker compose) - version too old 
    - curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg   
    - echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
    - apt-get update -y
    - apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
    - eval $(ssh-agent -s)
    - echo "$TOMCAT_SERVER_SSH_KEY" | tr -d '\r' | ssh-add -
    - ls -la
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan $TOMCAT_SERVER_IP >> ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts
    - echo $TOMCAT_SERVER_SSH_KEY
    # eventually not needed
    - echo $TOMCAT_SERVER_SSH_KEY >  ~/.ssh/id_rsa
    - chmod 600 ~/.ssh/id_rsa 

   script:
     - echo 'Deploying wordpress'
     - cd cms
     - export DOCKER_HOST="ssh://root@$TOMCAT_SERVER_IP"
     - docker info
     - docker container ls
     - docker compose up -d

```




```
docker compose up -d
```
