---
layout: post
title:  "Unpublished template"
published: false
---


```bash
brew cask install fly

fly -t wings login --concourse-url https://wings.pivotal.io --team-name PCFS

fly targets
fly -t wings pipelines

fly -t wings set-pipeline -p snap -c ci/pipeline.yml --load-vars-from ci/pipeline_params.yml
fly -t wings get-pipeline -p snap > pipeline.out

```
