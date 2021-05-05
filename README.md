# jenkins-playbook

## installation

jenkins installation: https://www.jenkins.io/download/lts/macos/

jenkins tutorial: https://www.jenkins.io/doc/tutorials/

```sh
# 安装
brew install jenkins-lts

# 启动jenkins
brew services start jenkins-lts
# 或者直接
jenkins-lts

```

## pipeline scripts

现在有两种方式声明pipeline:

- [Declarative pipeline](https://www.jenkins.io/doc/book/pipeline/#declarative-pipeline-fundamentals),声明式pipeline。jenkins提供的最新的功能
- [Scripted pipeline](https://www.jenkins.io/doc/book/pipeline/#scripted-pipeline-fundamentals)

上面两种DSLs([domain-specified language](https://en.wikipedia.org/wiki/Domain-specific_language))都可以用来定义jenkins pipeline。后者原本就有，通过[groovy脚本](http://groovy-lang.org/semantics.html)编写

所有的definition都会放在Jenkinsfile里面。

### Declarative pipeline

声明式语法

```jenkins

pipeline {
    agent any #1
    stages {
        stage('Build') {  #2
            steps {
                // #3
            }
        }
        stage('Test') { #4
            steps {
                // #5
            }
        }
        stage('Deploy') { #6
            steps {
                // #7
            }
        }
    }
}
```

- #1 在任何[agent](https://www.jenkins.io/doc/book/pipeline/syntax#agent)上面执行该pipeline或者其内定义的任意一个阶段
- #2 定义了名为Build的阶段
- #3 执行和Build阶段相关的任务或者脚本
- #4 定义名为Test的阶段
- #5 执行和test阶段相关的任务或者脚本
- #6 定义名为Deploy的阶段
- #7 执行和Deploy相关的任务或者脚本

需要注意的是：

- 声明式pipeline中，所有的任务计划都是封装在pipeline block中的
- stages模块，steps模块，在声明式和脚本式pipeline中都是可用的，并不是声明式独有。steps模块会锁住agents和Workspaces
- agent是声明式pipeline中独有的，在脚本pipeline中没有，Jenkins识别到agent关键字即为整个pipeline，分配一个executor或者一个node，且分配一个workspace


### scripted pipeline

```jenkins

node {   #1 
    stage('Build') { #2 
        sh 'make' #3
    }
    stage('Test') { 
        sh 'make check'
        junit 'reports/**/*.xml' #4
    }
    stage('Deploy') { 
        // 
    }
}

```

- #1 在任何可用的agent上，执行该pipeline或其内部定义的任何阶段，在脚本式语法中，node == 声明式语法中的agent
- #2 定义名为Build的阶段
- #3 执行与Build阶段相关的任务计划或者脚本，在这里是一个sh类型的pipeline step，sh是pipeline提供的，来执行shell命令的step
- #4 junit是另外一个pipeline step，由junit plugin提供

需要注意的是：

- 脚本式pipeline中，通过封装一个或者多个node模块，来构成整个pipeline，但是不是必须的，也就是说你可以不通过定义node来编写你的pipeline，前提是pipeline足够简单
- node模块做以下几件事情：1.在jenkins queue中增加一个item，来执行node模块内的steps,只要安排上了，只要jenkins executor空闲下来，就会执行steps
- 每一个node模块都会创建一个workspace(从属于该pipeline)，如果jenkins配置不自动清理workspace，那么跑完jenkins pipeline之后，可能会有workspace残留



## 构建pipeline

- 通过[blue ocean](https://www.jenkins.io/doc/book/blueocean/)来构建整个pipeline,相当于一整套流程化ui，以导航方式一步步帮助用户定义一个新的pipeline。
- 通过[classic ui](https://www.jenkins.io/doc/book/pipeline/getting-started/#through-the-classic-ui),由jenkins自己提供，通过new item一步步向下操作，然后手动定义jenkinsfile
- 一般情况下，项目不会是简单的jenkinsfile，一般都会通过github等source control system存储起来定期维护。所以Jenkins一般在定义pipeline的时候允许以in scm的方式，读取远程jenkinsfile。流程为：ide编写jenkinsfile ==> git hub commit ==> jenkins in scm pull the git hub jenkinsfile ==> run and build

### in scm

**注意：**在pipeline >> Definition栏目中，选择scm.

