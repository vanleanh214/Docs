## NHUNG LUU Y KHI VIET GITLAB-CI
**1. Setup configure**

- Add ssh-key id-xxx.pub of PC-dev & PC-Devops to Gitlab Portal

```bash
Icon User -> Preference -> User Settings -> SSH Keys
```

**Function:** *Support Dev pull & push code to Gitlab*

- Gitlab runner can use docker of host server

```bash
evg-user@jenkins-lab:~$ cat /home/evg-user/.gitlab-runner/config.toml
concurrent = 1
check_interval = 0
shutdown_timeout = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "evg-share"				-> *Name of runner*
  url = "http://gitlab.evgcloud.local"		-> *Domain of gitlab portal*(can use fix host or domain public)
  id = 1
  token = "glrt-stxvWz8Lu2mQr4FcV1vz"
  token_obtained_at = 2026-01-20T03:57:32Z
  token_expires_at = 0001-01-01T00:00:00Z
  executor = "docker"				-> *Method excute: docker(recommand) / shell /....*
  [runners.cache]
    MaxUploadedArchiveSize = 0
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "docker:24.0.5"			-> *Image defind while install runner*(can use latest)
    privileged = true				-> *true*(default is false)
    volumes = [
          "/var/run/docker.sock:/var/run/docker.sock",	-> *Mount volume of socket docker host outside*
            "/cache"
    ]
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    #volumes = ["/cache"]
    extra_hosts = ["gitlab.evgcloud.local:123.30.233.68", "registry.gitlab.evgcloud.local:123.30.233.68"]	-> *Fix host in container runner*(if using domain not public)
    shm_size = 0
    network_mtu = 0
```
- git clone git@gitlab.evgcloud.local:shoseshop/shoseshop.git

**Note:** clone code from Gitlab to PC Local

**2. Variable**

```bash
Project -> Setting -> CI/CD -> Variables
```
Note: Khai bao thong tin SSH cua cac server DEV & DEPLOY (ssh-ip, ssh-user, ssh-private-key) de con runner truy cap dc toi cac host.
      Cac bien nay khac voi cac bien trong moi truong o server deploy va dev.

**3. Example**

Example 1: build multi image (for dev)

```bash
stages:
  - build
  - deploy
  - notify_success
  - notify_failure
variables:
  - 
.build_template:
  before_script:
    - docker login $CI_REGISTRY -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD
  script:
    - docker pull $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.cms.latest || true
    - docker pull $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.api.latest || true
    - docker pull $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.teacherdev.latest || true
    - docker build --cache-from $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.cms.latest -t $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.$CI_COMMIT_SHORT_SHA.cms -t $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.cms.latest -f Dockerfile.cms .
    - docker build --cache-from $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.api.latest -t $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.$CI_COMMIT_SHORT_SHA.api -t $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.api.latest -f Dockerfile.api .
    - docker build --cache-from $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.teacherdev.latest -t $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.$CI_COMMIT_SHORT_SHA.teacherdev -t $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.teacherdev.latest -f Dockerfile.api .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.cms.latest
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.api.latest
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.teacherdev.latest
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.$CI_COMMIT_SHORT_SHA.cms
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.$CI_COMMIT_SHORT_SHA.api
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.$CI_COMMIT_SHORT_SHA.teacherdev
  after_script:
    - docker rmi $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.cms.latest
    - docker rmi $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.api.latest
    - docker rmi $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.teacherdev.latest
    - docker rmi $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.$CI_COMMIT_SHORT_SHA.cms
    - docker rmi $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.$CI_COMMIT_SHORT_SHA.api
    - docker rmi $CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.$CI_COMMIT_SHORT_SHA.teacherdev
    - docker rmi $(docker images -f "dangling=true" -q) || true


build:
  stage: build
  extends: .build_template
  only:
    - dev
  tags:
    - lms-evg

deploy-dev:
  stage: deploy
  variables:
    SSH_PATH: '/tmp/id_rs'
  image: alpine:latest
  before_script:
    - apk update && apk add openssh-client && apk add yq
    - echo "$SSH_KEY" > $SSH_PATH
    - chmod 600 $SSH_PATH
  script:
    - ssh -i $SSH_PATH -o StrictHostKeyChecking=no $SSH_USER@$SSH_SERVER_IP 'id'
    - ssh -i "$SSH_PATH" -o StrictHostKeyChecking=no "$SSH_USER@$SSH_SERVER_IP" \
      "echo '$CI_REGISTRY_PASSWORD' | docker login -u '$CI_REGISTRY_USER' --password-stdin '$CI_REGISTRY'"
    - ssh -i $SSH_PATH -o StrictHostKeyChecking=no $SSH_USER@$SSH_SERVER_IP 'docker compose -f /opt/truc-hoc-lieu-so-deployment/docker-compose.yml down -v'
    - ssh -i $SSH_PATH -o StrictHostKeyChecking=no $SSH_USER@$SSH_SERVER_IP "yq e '.services.\"app-cms\".image = \"$CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.$CI_COMMIT_SHORT_SHA.cms\"' /opt/truc-hoc-lieu-so-deployment/docker-compose.yml -i"
    - ssh -i $SSH_PATH -o StrictHostKeyChecking=no $SSH_USER@$SSH_SERVER_IP "yq e '.services.\"app-api\".image = \"$CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.$CI_COMMIT_SHORT_SHA.api\"' /opt/truc-hoc-lieu-so-deployment/docker-compose.yml -i"
    - ssh -i $SSH_PATH -o StrictHostKeyChecking=no $SSH_USER@$SSH_SERVER_IP "yq e '.services.\"app-teacher\".image = \"$CI_REGISTRY_IMAGE:$CI_COMMIT_BRANCH.$CI_COMMIT_SHORT_SHA.teacherdev\"' /opt/truc-hoc-lieu-so-deployment/docker-compose.yml -i"
    - ssh -i $SSH_PATH -o StrictHostKeyChecking=no $SSH_USER@$SSH_SERVER_IP 'docker compose -f /opt/truc-hoc-lieu-so-deployment/docker-compose.yml pull'
    - ssh -i $SSH_PATH -o StrictHostKeyChecking=no $SSH_USER@$SSH_SERVER_IP 'rm -rf /opt/truc-hoc-lieu-so-deployment/modules'
    - ssh -i $SSH_PATH -o StrictHostKeyChecking=no $SSH_USER@$SSH_SERVER_IP 'docker compose -f /opt/truc-hoc-lieu-so-deployment/docker-compose.yml up -d'
    - ssh -i $SSH_PATH -o StrictHostKeyChecking=no $SSH_USER@$SSH_SERVER_IP 'docker compose -f /opt/truc-hoc-lieu-so-deployment/docker-compose.yml ps'
  after_script:
    - ssh -i $SSH_PATH -o StrictHostKeyChecking=no $SSH_USER@$SSH_SERVER_IP 'docker image prune -a -f'
    - echo "Removing All Unused Images"
  only:
    - dev
  tags:
    - lms-evg

notify_success:
  image: alpine
  stage: notify_success
  before_script:
    - apk add --no-cache curl jq
  variables:
    CHAT_ID: "-4938393909"
    TOKEN_BOT: "7261921713:AAF2r6z_Yp8ehWV1k4a8JczxG_JXB_zhm0Y"
  script:
    - |
      curl -X POST -H "Content-Type: application/json" -d "{\"chat_id\": \"$CHAT_ID\", \"text\": \"✅ ${CI_PROJECT_NAME} pipeline succeed on branch ${CI_COMMIT_REF_NAME} of ${CI_COMMIT_MESSAGE}\"}" https://api.telegram.org/bot$TOKEN_BOT/sendMessage
  when: on_success
  only:
    - dev
  tags:
    - lms-evg


notify_failure:
  image: alpine
  stage: notify_failure
  before_script:
    - apk add --no-cache curl jq
  variables:
    CHAT_ID: "-4938393909"
    TOKEN_BOT: "7261921713:AAF2r6z_Yp8ehWV1k4a8JczxG_JXB_zhm0Y"
  script:
    - |
      curl -X POST -H "Content-Type: application/json" -d "{\"chat_id\": \"$CHAT_ID\", \"text\": \" ❌ Project: $CI_PROJECT_NAME $JOB_NAME false on branch ${CI_COMMIT_REF_NAME} of ${CI_COMMIT_MESSAGE} \"}" https://api.telegram.org/bot$TOKEN_BOT/sendMessage
  when: on_failure
  allow_failure: false
  only:
    - dev
  tags:
    - lms-evg
```

Example 2: Build single image (for dev)

```bash
variables:
  PROJECT_DIR: "/home/evg-user/shoseshop"
  IMAGE_NAME: vanleanh202/shoseshop
stages:
  - build
  - deploy

build_dev:
  stage: build
  before_script:
    - echo "$TOKEN" | docker login -u "$USERNAME" --password-stdin

  script:
    - docker pull $IMAGE_NAME:$CI_COMMIT_BRANCH-$CI_COMMIT_SHORT_SHA || true
    - docker build --cache-from $IMAGE_NAME:$CI_COMMIT_BRANCH-$CI_COMMIT_SHORT_SHA -t $IMAGE_NAME:$CI_COMMIT_BRANCH-$CI_COMMIT_SHORT_SHA -t $IMAGE_NAME:$CI_COMMIT_BRANCH-latest -f Dockerfile .
    - docker push $IMAGE_NAME:$CI_COMMIT_BRANCH-$CI_COMMIT_SHORT_SHA
    - docker push $IMAGE_NAME:$CI_COMMIT_BRANCH-latest

  after_script:
    - docker rmi $IMAGE_NAME:$CI_COMMIT_BRANCH-$CI_COMMIT_SHORT_SHA
    - docker rmi $IMAGE_NAME:$CI_COMMIT_BRANCH-latest

  only:
    - dev
  tags:
    - evg-share

deploy_dev:
  stage: deploy
  image: alpine:latest
  variables:
    SSH_PATH: '/tmp/id_private'

  before_script:
    - apk add --no-cache openssh-client && apk add yq
    - echo "$SSH_PRIVATE_KEY_DEV" > $SSH_PATH
    - chmod 600 $SSH_PATH

  script:
    - echo "1.LOGIN DOCKER REPO"
    - ssh -i $SSH_PATH -o StrictHostKeyChecking=no $SSH_USER_DEV@$SSH_IP_DEV "echo '$TOKEN' | docker login -u '$USERNAME' --password-stdin"
    - ssh -i $SSH_PATH -o StrictHostKeyChecking=no $SSH_USER_DEV@$SSH_IP_DEV "cd $PROJECT_DIR && git reset --hard origin/dev && git pull origin dev"
    - ssh -i $SSH_PATH -o StrictHostKeyChecking=no $SSH_USER_DEV@$SSH_IP_DEV "cd $PROJECT_DIR && yq e '.services.springboot.image = \"$IMAGE_NAME:$CI_COMMIT_BRANCH-$CI_COMMIT_SHORT_SHA\"' $PROJECT_DIR/docker-compose.yml -i"
    - ssh -i $SSH_PATH -o StrictHostKeyChecking=no $SSH_USER_DEV@$SSH_IP_DEV "cd "$PROJECT_DIR" && docker compose pull"
    - ssh -i $SSH_PATH -o StrictHostKeyChecking=no $SSH_USER_DEV@$SSH_IP_DEV "cd "$PROJECT_DIR" && docker compose up -d"
    - ssh -i $SSH_PATH -o StrictHostKeyChecking=no $SSH_USER_DEV@$SSH_IP_DEV 'docker ps -a'

  after_script:
    - ssh -i $SSH_PATH -o StrictHostKeyChecking=no $SSH_USER_DEV@$SSH_IP_DEV 'docker image prune -a -f'
  only:
    - dev
  tags:
    - evg-share
```
