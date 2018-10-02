# Spring Boot "Microservice" Example Project

This is a sample Java / Maven / Spring Boot (version 1.5.6) application that can be used as a starter for creating a microservice complete with built-in health check, metrics and much more. I hope it helps you.

## How to Run 

This application is packaged as a war which has Tomcat 8 embedded. No Tomcat or JBoss installation is necessary. You run it using the ```java -jar``` command.

* Clone this repository 
* Make sure you are using JDK 1.8 and Maven 3.x
* You can build the project and run the tests by running ```mvn clean package```
* Once successfully built, you can run the service by one of these two methods:
```
        java -jar -Dspring.profiles.active=test target/spring-boot-rest-example-0.5.0.war
or
        mvn spring-boot:run -Drun.arguments="spring.profiles.active=test"
```
* Check the stdout or boot_example.log file to make sure no exceptions are thrown

Once the application runs you should see something like this

```
2017-08-29 17:31:23.091  INFO 19387 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8090 (http)
2017-08-29 17:31:23.097  INFO 19387 --- [           main] Application        : Started Application in 22.285 seconds (JVM running for 23.032)
```

## About the Service

The service is just a simple hotel review REST service. It uses an in-memory database (H2) to store the data. You can also do with a relational database like MySQL or PostgreSQL. If your database connection properties work, you can call some REST endpoints defined in ```com.khoubyari.example.api.rest.hotelController``` on **port 8090**. (see below)

More interestingly, you can start calling some of the operational endpoints (see full list below) like ```/metrics``` and ```/health``` (these are available on **port 8091**)

You can use this sample service to understand the conventions and configurations that allow you to create a DB-backed RESTful service. Once you understand and get comfortable with the sample app you can add your own services following the same patterns as the sample service.
 
Here is what this little application demonstrates: 

* Full integration with the latest **Spring** Framework: inversion of control, dependency injection, etc.
* Packaging as a single war with embedded container (tomcat 8): No need to install a container separately on the host just run using the ``java -jar`` command
* Demonstrates how to set up healthcheck, metrics, info, environment, etc. endpoints automatically on a configured port. Inject your own health / metrics info with a few lines of code.
* Writing a RESTful service using annotation: supports both XML and JSON request / response; simply use desired ``Accept`` header in your request
* Exception mapping from application exceptions to the right HTTP response with exception details in the body
* *Spring Data* Integration with JPA/Hibernate with just a few lines of configuration and familiar annotations. 
* Automatic CRUD functionality against the data source using Spring *Repository* pattern
* Demonstrates MockMVC test framework with associated libraries
* All APIs are "self-documented" by Swagger2 using annotations 

Here are some endpoints you can call:

### Get information about system health, configurations, etc.

```
http://localhost:8091/env
http://localhost:8091/health
http://localhost:8091/info
http://localhost:8091/metrics
```

### Create a hotel resource

```
POST /example/v1/hotels
Accept: application/json
Content-Type: application/json

{
"name" : "Beds R Us",
"description" : "Very basic, small rooms but clean",
"city" : "Santa Ana",
"rating" : 2
}

RESPONSE: HTTP 201 (Created)
Location header: http://localhost:8090/example/v1/hotels/1
```

### Retrieve a paginated list of hotels

```
http://localhost:8090/example/v1/hotels?page=0&size=10

Response: HTTP 200
Content: paginated list 
```

### Update a hotel resource

```
PUT /example/v1/hotels/1
Accept: application/json
Content-Type: application/json

{
"name" : "Beds R Us",
"description" : "Very basic, small rooms but clean",
"city" : "Santa Ana",
"rating" : 3
}

RESPONSE: HTTP 204 (No Content)
```
### To view Swagger 2 API docs

Run the server and browse to localhost:8090/swagger-ui.html

# About Spring Boot

Spring Boot is an "opinionated" application bootstrapping framework that makes it easy to create new RESTful services (among other types of applications). It provides many of the usual Spring facilities that can be configured easily usually without any XML. In addition to easy set up of Spring Controllers, Spring Data, etc. Spring Boot comes with the Actuator module that gives the application the following endpoints helpful in monitoring and operating the service:

**/metrics** Shows “metrics” information for the current application.

**/health** Shows application health information.

**/info** Displays arbitrary application info.

**/configprops** Displays a collated list of all @ConfigurationProperties.

**/mappings** Displays a collated list of all @RequestMapping paths.

**/beans** Displays a complete list of all the Spring Beans in your application.

**/env** Exposes properties from Spring’s ConfigurableEnvironment.

**/trace** Displays trace information (by default the last few HTTP requests).

### To view your H2 in-memory datbase

The 'test' profile runs on H2 in-memory database. To view and query the database you can browse to http://localhost:8090/h2-console. Default username is 'sa' with a blank password. Make sure you disable this in your production profiles. For more, see https://goo.gl/U8m62X

# Running the project with MySQL

This project uses an in-memory database so that you don't have to install a database in order to run it. However, converting it to run with another relational database such as MySQL or PostgreSQL is very easy. Since the project uses Spring Data and the Repository pattern, it's even fairly easy to back the same service with MongoDB. 

Here is what you would do to back the services with MySQL, for example: 

### In pom.xml add: 

```
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
```

### Append this to the end of application.yml: 

```
---
spring:
  profiles: mysql

  datasource:
    driverClassName: com.mysql.jdbc.Driver
    url: jdbc:mysql://<your_mysql_host_or_ip>/bootexample
    username: <your_mysql_username>
    password: <your_mysql_password>

  jpa:
    hibernate:
      dialect: org.hibernate.dialect.MySQLInnoDBDialect
      ddl-auto: update # todo: in non-dev environments, comment this out:


hotel.service:
  name: 'test profile:'
```

### Then run is using the 'mysql' profile:

```
        java -jar -Dspring.profiles.active=mysql target/spring-boot-rest-example-0.5.0.war
or
        mvn spring-boot:run -Drun.jvmArguments="-Dspring.profiles.active=mysql"
```

# Attaching to the app remotely from your IDE

Run the service with these command line options:

```
mvn spring-boot:run -Drun.jvmArguments="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005"
or
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 -Dspring.profiles.active=test -Ddebug -jar target/spring-boot-rest-example-0.5.0.war
```
and then you can connect to it remotely using your IDE. For example, from IntelliJ You have to add remote debug configuration: Edit configuration -> Remote.


On every pipeline execution, the code goes through the following steps:

1. Code is cloned from **Github** or Gogs, built, tested and analyzed for bugs and bad patterns
2. The JAR artifact is pushed to **Nexus** Repository manager
3. A container image (_bookstore:latest_) is built based on the _bookstore_ application JAR artifact
4. The _bookstore_ container image is deployed in a fresh new container in bookstore_dev project
5. If tests successful, the **bookstore_dev** image is tagged with the application version in the **bookstore_stage** project
6. The staged image is deployed in a fresh new container in the **bookstore_stage** project

The application used in this pipeline is a Spring Boot application which is available on **src** folder in this repository 

## Prerequisites
* 8+ GB memory
* redhat-openjdk18-openshift imagestreams imported to OpenShift (see Troubleshooting section for details)

### Start up an OpenShift cluster:

```
minishift addons enable xpaas
#Adjust memory and cpus based on your PC, macbook or Ubuntu configuration
minishift start --memory=10240 --cpus=4 --vm-driver=virtualbox
oc login -u developer
```

### Pre-pull the images to make sure the deployments go faster:

```
minishift ssh docker pull openshiftdemos/gogs:0.11.34
minishift ssh docker pull registry.centos.org/rhsyseng/sonarqube:latest
minishift ssh docker pull sonatype/nexus3:3.8.0
minishift ssh docker pull registry.access.redhat.com/openshift3/jenkins-2-rhel7
minishift ssh docker pull registry.access.redhat.com/openshift3/jenkins-slave-maven-rhel7
minishift ssh docker pull registry.access.redhat.com/jboss-eap-7/eap70-openshift
```

### Get OC environment value and add it shell
##### Execute the following command and follow instructions on screen for next steps
```
$ minishift oc-env
 <Execute output of this command to get access to oc environment>

```

## Automated Deploy on OpenShift (Not Preffered)
You can se the `scripts/provision.sh` script provided to deploy the entire demo:

  ```
  ./provision.sh --help
  ./provision.sh deploy --deploy-che --ephemeral
  ./provision.sh delete
  ```

## Manual Deploy on OpenShift
Create the following projects for CI/CD components, Dev and Stage environments:

  ```
  # Create Projects
  oc new-project bookstore-dev --display-name="Bookstore - Dev"
  oc new-project bookstore-stage --display-name="Bookstore - Stage"
  oc new-project cicd --display-name="cicd"

  # Grant Jenkins Access to Projects
  oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n bookstore-dev
  oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n bookstore-stage

  ```  
  if add-role-to-user step fails execute following step and repeat last step again
  ```
  oc login -u system:admin
  oc adm policy add-cluster-role-to-user cluster-admin <username>
  ```
  
Clone the the project and navigate to the folder
```
  git clone https://github.com/pavankjadda/BookStore.git --branch=<branch name>
  cd Bookstore
```

1. Deploy the demo with one of the following methods (not both):

  ```
  # Deploy Demo
  oc new-app -n cicd -f cicd-template.yaml

  (OR)
  
  # Deploy Demo woth Eclipse Che
  oc new-app -n cicd -f cicd-template.yaml --param=WITH_CHE=true
  ```
2. Use Github instead of gogs. Skip step 3 if you use Github or gitlab
3. Start Gogs Server
```
oc new-app -f templates/gogs-template.yaml --param=GOGS_VERSION=0.11.34   --param=HOSTNAME='gogs'  --param=SKIP_TLS_VERIFY=true`
```  
  if gogs fail in this step, install gogs and postgres db in 2 steps. Do this only if Gogs failed in above step

  ```
      a. Go to homepage and Add to Project --> deploy image --> look for image ' centos/postgresql-95-centos7 ' --> deploy. Make
         sure to configure the following values before deploying
            POSTGRESQL_USER = gogs
            POSTGRESQL_DATABASE = gogs
            POSTGRESQL_DATABASE = gogs
      b.Finish steps 4 and 5 before performing this as previous step takes time. Go to homepage and Add to Project --> deploy image --> look for image ' openshiftdemos/gogs ' --> deploy. Once
        done go to home page, click on route. In new window, enter host ip as pod ip (get it from applications --> pods --> postgres pod --> IP)
        Enter username and password as 'gogs'. This will connect gogs to Postgres DB started in previous step.
```
4. Start sonarqube
```
    oc new-app -f templates/sonarqube-template.yaml --param=SONARQUBE_VERSION=7.0 --param=SONAR_MAX_MEMORY=2Gi
```
5. Start nexus artifact repository, this may take a while
```
oc new-app -f templates/nexus3-template.yaml --param=NEXUS_VERSION=3.7.1 --param=MAX_MEMORY=2Gi
```


To use custom project names, change `cicd`, `dev` and `stage` in the above commands to
your own names and use the following to create the demo:

  ```shell
  oc new-app -n cicd -f cicd-template.yaml --param DEV_PROJECT=dev-project-name --param STAGE_PROJECT=stage-project-name
  ```

# Jenkinsfile
Following Jenkinsfile (this code located inside cicd-template.yaml file) contains steps to automate the build and deployment process. Please make changes to Jenkinsfile if you want to add/remove steps in future. Next step explains the same process in a manual way. 


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
                  openshift.set("probe dc/spring-boot-rest-example --liveness  --get-url=http://:8080/health --initial-delay-seconds=180 --failure-threshold=10 --period-seconds=10")
    
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
        stage('Promote to STAGE?') {
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
                  openshift.set("probe dc/spring-boot-rest-example --liveness  --get-url=http://:8080/health --initial-delay-seconds=180 --failure-threshold=10 --period-seconds=10")
                }
              }
            }
          }
        }
      }
    }




# Ignore this step (Wrote only for Understanding purpose) 

Jenkinsfile has same code but this step explains the steps one by one. Wrote this only for understanding purpose. Build Application from source code/get from Artifact Repository. This [article](https://access.redhat.com/documentation/en-us/red_hat_jboss_middleware_for_openshift/3/html-single/red_hat_java_s2i_for_openshift/index) explains
  the whole process. Here is a short version of it

## Source to Image (S2I) Build (Not recommended)

To run and configure the Java S2I for OpenShift image, use the OpenShift S2I process.

The S2I process for the Java S2I for OpenShift image works as follows:

Log into the OpenShift instance by running the following command and providing credentials.

    $ oc login

Create a new project.

    $ oc new-project <project-name>

Create a new application using the Java S2I for OpenShift image. <source-location> can be the URL of a git repository or a path to a local folder.

    $ oc new-app redhat-openjdk18-openshift~<source-location>
Get the service name.

    $ oc get service

Expose the service as a route to be able to use it from the browser. <service-name> is the value of NAME field from previous command output.

    $ oc expose svc/<service-name> --port=8080

Get the route.

    $ oc get route

Access the application in your browser using the URL (value of HOST/PORT field from previous command output).

## Binary Builds (Build from Jar, War file)
### To deploy existing applications on OpenShift, you can use the binary source capability.

Prerequisite:

Get the JAR application archive or build the application locally.
The example below uses the undertow-servlet quickstart.

Clone the source code.

        $ git clone https://github.com/jboss-openshift/openshift-quickstarts.git

Configure the Red Hat JBoss Middleware Maven repository.

Build the application.

        $ cd openshift-quickstarts/undertow-servlet/

        $ mvn clean package
        [INFO] Scanning for projects...
        ...
        [INFO]
        [INFO] ------------------------------------------------------------------------
        [INFO] Building Undertow Servlet Example 1.0.0.Final
        [INFO] ------------------------------------------------------------------------
        ...
        [INFO] ------------------------------------------------------------------------
        [INFO] BUILD SUCCESS
        [INFO] ------------------------------------------------------------------------
        [INFO] Total time: 1.986 s
        [INFO] Finished at: 2017-06-27T16:43:07+02:00
        [INFO] Final Memory: 19M/281M
        [INFO] ------------------------------------------------------------------------

Prepare the directory structure on the local file system.

Application archives in the deployments/ subdirectory of the main binary build directory are copied directly to the standard deployments folder of the image being built on OpenShift. For the application to deploy, the directory hierarchy containing the web application data must be correctly structured.

Create main directory for the binary build on the local file system and deployments/ subdirectory within it. Copy the previously built JAR archive to the deployments/ subdirectory:

#### ignore above two steps as we already did this through Jenkins file
    $ ls
    dependency-reduced-pom.xml  pom.xml  README  src  target

    $ mkdir -p ocp/deployments

    $ cp target/undertow-servlet.jar ocp/deployments/

##### Note
Location of the standard deployments directory depends on the underlying base image, that was used to deploy the application. See the following table:

Perform the following steps to run application consisting of binary input on OpenShift:

Log into the OpenShift instance by running the following command and providing credentials.

    $ oc login

Create a new project.

    $ oc new-project <bookstore>
(Optional) Identify the image stream for the particular image.

    $ oc get is -n openshift | grep ^redhat-openjdk | cut -f1 -d ' '
    redhat-openjdk18-openshift
Create new binary build, specifying image stream and application name.

    $ oc new-build --binary=true \
    --name=bookstore \
    --image-stream=redhat-openjdk18-openshift:1.3
    --> Found image c1f5b31 (2 months old) in image stream "openshift/redhat-openjdk18-openshift" under tag "latest" for "redhat-openjdk18-openshift"

    Java Applications
    -----------------
    Platform for building and running plain Java applications (fat-jar and flat classpath)
    Tags: builder, java

    * A source build using binary input will be created
    * The resulting image will be pushed to image stream "bookstore:latest"
    * A binary build was created, use 'start-build --from-dir' to trigger a new build

        --> Creating resources with label build=jdk-us-app ...
            imagestream "bookstore" created
            buildconfig "bookstore" created
        --> Success

Start the binary build. Instruct oc executable to use main directory of the binary build we created in previous step as the directory containing binary input for the OpenShift build.

    $ oc start-build bookstore --from-dir=./ocp --follow
    Uploading directory "ocp" as binary input for the build ...
    build "bookstore-1" started
    Receiving source from STDIN as archive ...
    ==================================================================
    Starting S2I Java Build .....
    S2I source build with plain binaries detected
    Copying binaries from /tmp/src/deployments to /deployments ...
    ... done
    Pushing image 172.30.197.203:5000/bookstore/bookstore:latest ...
    Pushed 0/6 layers, 2% complete
    Pushed 1/6 layers, 24% complete
    Pushed 2/6 layers, 36% complete
    Pushed 3/6 layers, 54% complete
    Pushed 4/6 layers, 71% complete
    Pushed 5/6 layers, 95% complete
    Pushed 6/6 layers, 100% complete
    Push successful

Create a new OpenShift application based on the build.

    $ oc new-app spring-boot-rest-example

Expose the service as route.

    $ oc get svc -o name

    $ oc expose svc/spring-boot-rest-example

Access the application.

Access the application in your browser using the URL http://spring-boot-rest-example-spring-boot-rest-example-stage.192.168.64.4.nip.io/env


# Questions and Comments: email.sameer@gmail.com

