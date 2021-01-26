---
title: "[Jenkins] Build using shell scripts"
excerpt: "Jenkins Build using shell scripts"
description: "Jenkins Build using shell scripts"
modified: 2020-05-25
categories: "Jenkins"
tags: [Jenkins, Build, Back-end, Server]

toc: true
toc_sticky: true

header:
  teaser: /assets/images/jenkins-logo.png
---

#  Build using shell scripts (참고자료..)

## 1. Version up the Xcode Project

### 1) git checkout and pull
- 프로젝트 해당 브랜치의 커밋 내용 모두 pull
```bash
  cd $PROJECT_PATH

  git stash
  git pull
  git checkout $branch
  git pull
```
### 2) Update appVerion and buildNumber in PAYCO.xcodeproj
#### a. Version up
- xcode 프로젝트 파일에서 기존의 앱버전과 빌드넘버를 파싱하여 각각 한 버전씩 업데이트
```bash
  if [ -z $version ];then
    version=$(egrep -o 'APP_VERSION = [0-9\.]*;' $XCODE_PROJECT_FILE_PATH | sort -u | sed -e "s/APP_VERSION = //g" | sed -e "s/;//g")
    version=$(echo $version | awk '{ split($0, arr, "."); arr[4]+=1; print arr[1]"."arr[2]"."arr[3]"."arr[4];}')
  fi

  if [ -z $build_number ];then
    build_number=$(egrep -o 'BUILD_NUMBER = [0-9\.]*;' $XCODE_PROJECT_FILE_PATH | sort -u | sed -e "s/BUILD_NUMBER = //g" | sed -e "s/;//g")
    (( build_number+=1 ))
  fi
```
#### b. Update in xcode projectfile
- 업데이트된 앱버전과 빌드넘버를 기존 xcode 프로젝트 파일에 수정반영
```bash
  echo "APP_VERSION : " $version
  sed -i "" "s/APP_VERSION = [0-9\.]*;/APP_VERSION = $version;/g" $XCODE_PROJECT_FILE_PATH

  echo "BUILD_NUMBER : " $build_number
  sed -i "" "s/BUILD_NUMBER = [0-9]*;/BUILD_NUMBER = $build_number;/g" $XCODE_PROJECT_FILE_PATH
```
### 3) git commit and push
- 업데이트된 버전내용을 깃에 커밋
```bash
  git add $XCODE_PROJECT_FILE_PATH
  git commit -m "<update version> $version $build_number"
  git push origin $branch
```

### 4) git log
- grep commit message : `<update version>`, `]`
```bash
git log --pretty="- %s" --since=$days"days ago" --grep="<update version>" --grep="]"
```

## 2. Build the Project (Using Jenkins API)

- 젠킨스의 REST API를 이용하여, 원격에서 쉘스크립트로 젠킨스 서버를 동작 시킴
- jekins API URL : $JENKINS_SERVER/api

### 1) Get jobs in a view of jenkins
- 젠킨스에 설정된 view 이름을 이용하여, view에 속한 job 리스트를 얻어옴
- view에 속한 job 리스트 api : `$JENKINS_SERVER/view/$jenkins_view/api/json`
```bash
  urls=($(curl -s $JENKINS_SERVER/view/$jenkins_view/api/json | python -m json.tool | awk '/url/ && /job/ {print $2}' | sed 's/\"//g'))

  for job_url in "${urls[@]}"; do
      url_path_array=($(echo $job_url | sed 's/\// /g'))
      job_name=${url_path_array[3]}

      if [ $branch ] # if need to change the branch name in config.xml of jenkins jobs
      then
        jenkins_build_edit_config_xml $job_name $job_url $branch
      fi
      jenkins_build_excute $job_name $job_url
  done
```
### 2) Update config.xml of each job (Change name of git branch) : jenkins_build_edit_config_xml
- 젠킨스 job에 설정 내용을 수정해야 할 경우 설정 .xml 파일을 다운로드 받아서, 로컬에서 수정 후 다시 해당 job에 반영
- 보통 다른 깃 브랜치를 이용하여 빌드 해야될 경우, job 설정에서 깃 브랜치명을 변경하는 경우 사용
#### a. Download config.xml
- job 설정 파일 .xml 로 다운로드
- job 설정 파일 api : `$job_url/config.xml`
```bash
  config_xml_url=$job_url/config.xml

  curl -s $config_xml_url > xml/$job_name.xml
```
#### b. Edit & Save new config.xml
- 설정파일에서 깃 브랜치이름 변경
```bash
  current_branch=$(grep "<name>.*<.name>" xml/$job_name.xml | sed -e "s/^.*<name/<name/" | cut -f2 -d">"| cut -f1 -d"<")

  sed -e "s/<name>$current_branch<\/name>/<name>$new_branch<\/name>/g" xml/$job_name.xml > xml/new/$job_name.xml
```
#### c. Upload new config.xml
- 수정된 설정 파일을 다시 해당하는 job에 업로드하여 적용
- job 설정 파일 업로드 api :  `POST, $job_url/config.xml`
```bash
  curl -X POST -H 'Content-Type: application/xml; charset=utf-8' $config_xml_url --data-binary @xml/new/$job_name.xml
```
### 3) Execute build : jenkins_build_excute
- 해당 job 원격 실행
- job 원격 실행 api : `$job_url/build`
```bash
  curl -s $job_url/build
```
