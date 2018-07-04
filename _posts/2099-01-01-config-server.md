---
layout: post
title:  "Unpublished template"
published: false
---


```

cf create-service p-config-server standard config-server -c '{"git": { "uri": "https://github.com/ynovytskyy/app-config.git" }}'


```

TOKEN=`curl -s https://p-spring-cloud-services.uaa.run.pivotal.io/oauth/token -u "p-config-server-ce811614-a588-4a47-896a-d92a0a89247a:dtEKeEqEgvyy" -d "grant_type=client_credentials" | jq -r .access_token`

echo $TOKEN

curl -v -k -H "Authorization: bearer $TOKEN" -H "Accept: application/json" https://config-42dc9988-f93a-4535-8563-b4f83021df77.cfapps.io/foo-default.properties
