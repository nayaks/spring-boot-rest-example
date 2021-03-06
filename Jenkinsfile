def version, mvnCmd = "mvn -s config/cicd-settings-nexus3.xml"
pipeline {
 agent {
  label 'maven'
 }
 stages {
  stage('Build App') {
   steps {
    git branch: 'master', url: 'https://github.com/nayaks/spring-boot-rest-example.git'
    script {
      def pom = readMavenPom file: 'pom.xml'
      version = pom.version
    }
    sh "${mvnCmd} install -DskipTests=true"
   }
  }
  stage('Test') {
   steps {
    sh "${mvnCmd} test"
    step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
   }
  }
  stage('Code Analysis') {
   steps {
    script {
     sh "${mvnCmd} sonar:sonar -Dsonar.host.url=http://sonarqube:9000 -DskipTests=true"
    }
   }
  }
  /*
  stage('Archive App') {
   steps {
    sh "${mvnCmd} deploy -DskipTests=true -P nexus3"
   }
  }*/
  stage('Create Image Builder') {
   when {
    expression {
     openshift.withCluster() {
      openshift.withProject(env.DEV_PROJECT) {
       return !openshift.selector("bc", "spring-boot-rest-example").exists();
      }
     }
    }
   }
   steps {
    script {
     openshift.withCluster() {
      openshift.withProject(env.DEV_PROJECT) {
       openshift.newBuild("--name=spring-boot-rest-example", "--image-stream=redhat-openjdk18-openshift:1.2", "--binary=true")
      }
     }
    }
   }
  }
  stage('Build Image') {
   steps {
    sh "rm -rf ocp && mkdir -p ocp/deployments"
    sh "pwd && ls -la target "
    sh "cp target/spring-boot-rest-example-*.jar ocp/deployments"
    script {
     openshift.withCluster() {
      openshift.withProject(env.DEV_PROJECT) {
       openshift.selector("bc", "spring-boot-rest-example").startBuild("--from-dir=./ocp","--follow", "--wait=true")
      }
     }
    }
   }
  }
  stage('Create DEV') {
   when {
    expression {
     openshift.withCluster() {
      openshift.withProject(env.DEV_PROJECT) {
       return !openshift.selector('dc', 'spring-boot-rest-example').exists()
      }
     }
    }
   }
   steps {
    script {
     openshift.withCluster() {
      openshift.withProject(env.DEV_PROJECT) {
       def app = openshift.newApp("spring-boot-rest-example:latest")
       app.narrow("svc").expose();
       //http://localhost:8080/actuator/health
       openshift.set("probe dc/spring-boot-rest-example --readiness --get-url=http://:8080/health --initial-delay-seconds=30 --failure-threshold=10 --period-seconds=10")
       openshift.set("probe dc/spring-boot-rest-example --liveness --get-url=http://:8080/health --initial-delay-seconds=180 --failure-threshold=10 --period-seconds=10")
       def dc = openshift.selector("dc", "spring-boot-rest-example")
       while (dc.object().spec.replicas != dc.object().status.availableReplicas) {
         sleep 10
       }
       openshift.set("triggers", "dc/spring-boot-rest-example", "--manual")
      }
     }
    }
   }
  }
  stage('Deploy DEV') {
   steps {
    script {
     openshift.withCluster() {
      openshift.withProject(env.DEV_PROJECT) {
        openshift.selector("dc", "spring-boot-rest-example").rollout().latest();
      }
     }
    }
   }
  }
  stage('Promote to STAGE') {
   steps {
    script {
     openshift.withCluster() {
      openshift.tag("${env.DEV_PROJECT}/spring-boot-rest-example:latest", "${env.STAGE_PROJECT}/spring-boot-rest-example:${version}")
     }
    }
   }
  }
  stage('Deploy STAGE') {
   steps {
    script {
     openshift.withCluster() {
      openshift.withProject(env.STAGE_PROJECT) {
       if (openshift.selector('dc', 'spring-boot-rest-example').exists()) {
        openshift.selector('dc', 'spring-boot-rest-example').delete()
        openshift.selector('svc', 'spring-boot-rest-example').delete()
        openshift.selector('route', 'spring-boot-rest-example').delete()
       }
       openshift.newApp("spring-boot-rest-example:${version}").narrow("svc").expose()
       openshift.set("probe dc/spring-boot-rest-example --readiness --get-url=http://:8080/health --initial-delay-seconds=30 --failure-threshold=10 --period-seconds=10")
       openshift.set("probe dc/spring-boot-rest-example --liveness --get-url=http://:8080/health --initial-delay-seconds=180 --failure-threshold=10 --period-seconds=10")
      }
     }
    }
   }
  }
 }
}