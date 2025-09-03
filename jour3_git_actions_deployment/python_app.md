# Version slim-buster
```
git clone https://gitlab.com/thisalwijayakumara/gitlab-cicd-crash-course.git
```
```
ls
```
```
cd gitlab-cicd-crash-course/
```
```
ls
```
```
docker build -t test .
```
```
docker run -td -p 5000:5000 test
```
* DockerFile
```
FROM python:3.9-slim-buster

LABEL Name="Python Flask Demo App" Version=1.4.2
LABEL org.opencontainers.image.source = "https://github.com/benc-uk/python-demoapp"

ARG srcDir=src
WORKDIR /app
COPY $srcDir/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY $srcDir/run.py .
COPY $srcDir/app ./app

EXPOSE 5000

CMD ["gunicorn", "-b", "0.0.0.0:5000", "run:app"]
```
* .gitlab-ci.yml
```
variables:
    IMAGE_NAME: thisal1995/gitlab
    IMAGE_TAG:  python-app-1.0

stages:
    -   test
    -   build
    -   deploy

run_tests:
    stage:  test
    image:  python:3.9-slim-bullseye
    before_script:
        -   apt-get update  &&  apt-get install  make
    script:
        -   make test

build_image:
    stage:  build
    image:  docker:28.4.0-rc.1-cli
    services:
        -   docker:28.4.0-rc.1-dind
    variables:
        DOCKER_TLS_CERTDIR: "/certs"
    before_script:
        -   docker  login   -u  $REGISTRY_USER  -p  $REGISTRY_PASS
    script:
        -   docker  build   -t  $IMAGE_NAME:$IMAGE_TAG .
        -   docker  push  $IMAGE_NAME:$IMAGE_TAG  


deploy:
    stage: deploy
    before_script:
        -   chmod 400 $SSH_KEY
    script:
        -   ssh -o  StrictHostKeyChecking=no    -i  $SSH_KEY ec2-user@13.126.214.177 "
            docker  login   -u  $REGISTRY_USER  -p  $REGISTRY_PASS &&
            docker ps -aq | xargs --no-run-if-empty docker stop &&
            docker ps -aq | xargs --no-run-if-empty docker rm &&
            docker run -dt -p 5000:5000 $IMAGE_NAME:$IMAGE_TAG"
```
# Version python-3.11-slim
Version latest : https://gitlab.com/Bigdawg3442/gitlab-cicd-crash-course-1.git
* Makefile
```
# Used by `image`, `push` & `deploy` targets, override as required
IMAGE_REG ?= ghcr.io
IMAGE_REPO ?= benc-uk/python-demoapp
IMAGE_TAG ?= latest

# Used by `deploy` target, sets Azure webap defaults, override as required
AZURE_RES_GROUP ?= temp-demoapps
AZURE_REGION ?= uksouth
AZURE_SITE_NAME ?= pythonapp-$(shell git rev-parse --short HEAD)

# Used by `test-api` target
TEST_HOST ?= localhost:5000

# Don't change
SRC_DIR := src

.PHONY: help lint lint-fix image push run deploy undeploy clean test-api .EXPORT_ALL_VARIABLES
.DEFAULT_GOAL := help

help:  ## 💬 This help message
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-20s\033[0m %s\n", $$1, $$2}'

lint: venv  ## 🔎 Lint & format, will not fix but sets exit code on error 
	. $(SRC_DIR)/.venv/bin/activate \
	&& black --check $(SRC_DIR) \
	&& flake8 src/app/ && flake8 src/run.py

lint-fix: venv  ## 📜 Lint & format, will try to fix errors and modify code
	. $(SRC_DIR)/.venv/bin/activate \
	&& black $(SRC_DIR)

image:  ## 🔨 Build container image from Dockerfile 
	docker build . --file build/Dockerfile \
	--tag $(IMAGE_REG)/$(IMAGE_REPO):$(IMAGE_TAG)

push:  ## 📤 Push container image to registry 
	docker push $(IMAGE_REG)/$(IMAGE_REPO):$(IMAGE_TAG)

run: venv  ## 🏃 Run the server locally using Python & Flask
	. $(SRC_DIR)/.venv/bin/activate \
	&& python src/run.py

deploy:  ## 🚀 Deploy to Azure Web App 
	az group create --resource-group $(AZURE_RES_GROUP) --location $(AZURE_REGION) -o table
	az deployment group create --template-file deploy/webapp.bicep \
		--resource-group $(AZURE_RES_GROUP) \
		--parameters webappName=$(AZURE_SITE_NAME) \
		--parameters webappImage=$(IMAGE_REG)/$(IMAGE_REPO):$(IMAGE_TAG) -o table 
	@echo "### 🚀 Web app deployed to https://$(AZURE_SITE_NAME).azurewebsites.net/"

undeploy:  ## 💀 Remove from Azure 
	@echo "### WARNING! Going to delete $(AZURE_RES_GROUP) 😲"
	az group delete -n $(AZURE_RES_GROUP) -o table --no-wait

test: src/.venv/touchfile  ## 🎯 Unit tests for Flask app
	$(SRC_DIR)/.venv/bin/python -m pip install -r $(SRC_DIR)/requirements.txt
	$(SRC_DIR)/.venv/bin/python -m pytest src/app/tests/ -v --disable-warnings

test-report: venv  ## 🎯 Unit tests for Flask app (with report output)
	. $(SRC_DIR)/.venv/bin/activate \
	&& pytest -v --junitxml=test-results.xml

test-api: .EXPORT_ALL_VARIABLES  ## 🚦 Run integration API tests, server must be running 
	cd tests \
	&& npm install newman \
	&& ./node_modules/.bin/newman run ./postman_collection.json --env-var apphost=$(TEST_HOST)

clean:  ## 🧹 Clean up project
	rm -rf $(SRC_DIR)/.venv
	rm -rf tests/node_modules
	rm -rf tests/package*
	rm -rf test-results.xml
	rm -rf $(SRC_DIR)/app/__pycache__
	rm -rf $(SRC_DIR)/app/tests/__pycache__
	rm -rf .pytest_cache
	rm -rf $(SRC_DIR)/.pytest_cache

# ============================================================================

venv: $(SRC_DIR)/.venv/touchfile

$(SRC_DIR)/.venv/touchfile: $(SRC_DIR)/requirements.txt
	python -m venv $(SRC_DIR)/.venv
	. $(SRC_DIR)/.venv/bin/activate; pip install -Ur $(SRC_DIR)/requirements.txt
	touch $(SRC_DIR)/.venv/touchfile
```

* DockerFile
```
FROM python:3.11-slim

LABEL Name="Python Flask Demo App" Version=1.4.2
LABEL org.opencontainers.image.source = "https://github.com/benc-uk/python-demoapp"

ARG srcDir=src
WORKDIR /app
COPY $srcDir/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY $srcDir/run.py .
COPY $srcDir/app ./app

EXPOSE 5000

CMD ["gunicorn", "-b", "0.0.0.0:5000", "run:app"]
```

* .gitlab-cicd.yml
```
variables:
    IMAGE_NAME: olatomiwa17/demo-app
    IMAGE_TAG: python-app-1.0

stages:
  - test
  - build

run_tests:
  stage: test
  image: python:3.11-slim
  before_script:
    - apt-get update && apt-get install -y make git python3-dev gcc build-essential
    - pip install --upgrade pip
  script:
    - make test

build_image:
  stage: build
  image: docker:28.3.3-cli
  services:
    - docker:28.3.3-dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - docker login -u $REGISTRY_USER -p $REGISTRY_PASS
  script:
    - docker build -t $IMAGE_NAME:$IMAGE_TAG .
    - docker push $IMAGE_NAME:$IMAGE_TAG
```


```
ls
```
```
cd gitlab-cicd-crash-course/
```
```
ls
```
```
docker build -t test .
```
```
docker run -td -p 5000:5000 test
```

* DockerFile

