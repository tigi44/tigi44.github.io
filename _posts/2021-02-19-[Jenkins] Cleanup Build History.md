---
title: "[Jenkins] Cleanup Build History"
excerpt: "Jenkins Cleanup Build History"
description: "Jenkins Cleanup Build History"
modified: 2021-02-19
categories: "Jenkins"
tags: [Jenkins, Cleanup Build]

header:
  teaser: /assets/images/jenkins-teaser.png
---

## 1. Using Jenkins Script Console

- Manage Jenkins > Script Console

![ScriptConsole](/assets/images/post/jenkins/scriptconsole.png)

### All Jobs

```
Jenkins.instance.getAllItems().each() { job ->
  job.builds.each() { build ->
    build.delete()
  }

  job.updateNextBuildNumber(1)
}
```

### Specific Job

```
job = Jenkins.instance.getItemByFullName("job_name")

job.builds.each() { build ->
  build.delete()
}

job.updateNextBuildNumber(1)
```

## 2. Using Shell script

### Remove all builds directory
```shell
$ /your_jenkins_home/jobs> rm -rf */builds/*
```

### Reload Jenkins
- Manage Jenkins > Reload Configuration from Disk

![reloadmenu](/assets/images/post/jenkins/reloadmenu.png)
