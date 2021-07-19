---
layout:     post
title:      "Gitops In Kubernetes With Jenkins And ArgoCD"
subtitle:   "Gitops At Scale With Jenkins And ArgoCD On Kubernetes"
date:       2021-07-19 18:00:00
author:     "Thaung Htike Oo"
header-img: img/black.jpg
catalog: true
tags:
    - Kubernetes
    - Jenkins
    - CICD
---    

<h2> Introduction </h2>

ကျွန်တော်ဒီနေ့တော့ GitOps အကြောင်းကိုနားလည်သလောက် ရှင်းပြသွားမှာဖြစ်ပါတယ်။ ပြီးရင် demo တစ်ခုနဲ့စမ်းပြပါမယ်။ ဒီနေ့အတွက်ခေါင်းစဥ် ကိုတော့ 'gitops in kubernetes with jenkins and argocd' လို့ပေးထားပါတယ်။ ဒီနေ့ demo မှာ jenkins ကို docker image တွေ build လုပ်ဖို့၊ pushလုပ်ဖို့ စသည်တို့အတွက် CI tool အဖြစ်သုံးမှာဖြစ်ပြီး argocd ကိုတော့ kubernetes အတွက် continuous deployments တွေလုပ်ဖို့သုံးမှာဖြစ်ပါတယ်။
Jenkins install လုပ်နည်းကို ရှင်းပြပြီးပြီ ဆိုတော့ ပထမဆုံးအနေနဲ့ jenkins pipeline တစ်ခုကို create ပါမယ်။ jenkins install လုပ်ပုံကို အောက်က link မှာဖတ်နိုင်ပါတယ်။

[install_jenkins](https://thaunggye.github.io/2021-07-07-jenkins)

<h2> Prequities </h2>

<ul>
    <li> Jenkins Server </li>
    <li> Kubernetes Cluster </li>
</ul>    

<h2> Create a Pipeline </h2>

Jenkins dashboard ထဲရောက်သွားပြီဆိုရင် New Item ကနေ pipeline project တစ်ခု create ပါမယ်။ name ကိုတော့ djano_app လို့ပေးထားပါတယ်။ docker နဲ့ဆိုင်တာတွေကို jenkins မှာလုပ်မှာဆိုတော့ Manage Plugins ထဲကနေ docker plugins ကို install ပေးရပါမယ်။ 

![dapp]()

github project မှာ demo အတွက်သုံးမယ်ံ repo ကိုထည့်ပါမယ်။

![p1]()

pipeline script ထဲမှာတော့ docker image ကို build လုပ်ဖို့၊ push လုပ်ဖို့အတွက်ရေးပေးမှာဖြစ်ပါတယ်။ script ကို အောက်မှာကြည့်နိုင်ပါတယ်။ ပထမအဆင့်မှာ git repo နဲ့ branch ကိုထည့်မယ်၊ ပြီးရင် docker image ကို build မယ် name က mcapp ၊ tag က 1.$BUILD_NUMBER ဆိုပြီးပေးမယ်။ ပြီးရင် docker login လုပ်ပြီး docker hub ထဲကို push လိုက်ပါမယ်။

```yaml
pipeline {
    agent any
    
    environment {
        DOCKER_USER = 'tho861998'
        DOCKER_REGISTRY = 'docker.io'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/thaunggyee/django_mc_app.git'
            }
        }
        stage('docker image building') {
            steps {
                sh 'docker build -t tho861998/mcapp:1.${BUILD_NUMBER} .'
            }
        }
        stage('docker login') {
            steps {
                sh 'docker login -u $DOCKER_USER -p $DOCKER_PASSWORD $DOCKER_REGISTRY'    
            }
        }
        stage('docker push') {
            steps {
                sh 'docker push tho861998/mcapp:1.${BUILD_NUMBER}'
            }
        }
    }    
}
```
ပြီးသွားရင်တော့ save and apply ကိုနှိပ်လိုက်ပါ။ ဒါဆိုရင် pipeline ကို build လုပ်လို့ရပါပြီ။ ဘေးက BUILD NOW ဆိုတာကိုနှိပ်လိုက်ရင် pipeline တစ်ခု build နေတာကိုအခုလိုတွေ့ရမှာဖြစ်ပါတယ်။

![finish]()

build တာပြီးသွားပြီဆိုရင် docker hub ထဲမှာ mcapp ဆိုပြီး image တစ်ခုတွေ့ရမှာဖြစ်ပါတယ်။ tag ကတော့ build တစ်ခုပဲရှိသေးတော့ 1.1 တစ်ခုပဲရှိပါဦးမယ်။

![mcapp1]()
