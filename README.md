# Camel/XML Swagger Contract First

## Requirements

- [Apache Maven 3.x](http://maven.apache.org)
- [MySQL 5.7.18](https://www.mysql.com/oem/)
  - [Docker Image](https://hub.docker.com/r/mysql/mysql-server/)

## Preparing

Install and run MySQL [https://dev.mysql.com/doc/refman/5.7/en/installing.html]

_Note: For my tests, I chose to run the docker image [https://hub.docker.com/r/mysql/mysql-server/]. You can run it using the command `docker run --name mysql -e MYSQL_DATABASE=example -e MYSQL_ROOT_PASSWORD=Abcd1234 -e MYSQL_ROOT_HOST=172.17.0.1 -p 3306:3306 -d mysql/mysql-server:5.7`. You can then connect and run SQL statements using the command `docker exec -it mysql mysql -uroot -p`._

Build the project source code

```
$ cd $PROJECT_ROOT
$ mvn clean install
```

## Running the example standalone

```
$ cd $PROJECT_ROOT
$ mvn spring-boot:run
```

## Testing

To pull up the Swagger/OpenAPI spec:

```
$ curl -X GET -H 'Accept: application/json' 'http://localhost:8080/services/camel/swagger.json'
```

To get a pet by ID:

```
$ curl -X GET -H 'Accept: application/json' 'http://localhost:8080/services/camel/v2/pet/1'
```

To get a list of pets by status:

```
$ curl -X GET -H 'Accept: application/json' 'http://localhost:8080/services/camel/v2/pet/findByStatus?status=available'
```

To get a list of pets by tags:

```
$ curl -X GET -H 'Accept: application/json' 'http://localhost:8080/services/camel/v2/pet/findByTags?tags=Sphynx&tags=Evil'
```

To add a pet:

```
curl -X POST -H 'Accept: application/json' -H 'Content-Type: application/json' -d '{ "category" : { "name" : "bird" }, "name" : "Toucan Sam", "photoUrls" : [ "https://www.thedailymeal.com/sites/default/files/story/2017/toucansam%20crop.jpg" ], "tags" : [ { "name" : "Toucan" } ], "status" : "AVAILABLE" }' 'http://localhost:8080/services/camel/v2/pet'
```

## Running the example in OpenShift

It is assumed that:

- OpenShift platform is already running, if not you can find details how to [Install OpenShift at your site](https://docs.openshift.com/container-platform/3.6/install_config/index.html).
- Your system is configured for Fabric8 Maven Workflow, if not you can find a [Get Started Guide](https://access.redhat.com/documentation/en-us/red_hat_jboss_fuse/6.3/html-single/fuse_integration_services_2.0_for_openshift/index)

Create a new project:

```
$ oc new-project demo-fuse
```

Create the [ServiceAccount](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/):

```
$ oc create -f src/main/kube/serviceaccount.yml
```

Create the [ConfigMap](https://kubernetes.io/docs/user-guide/configmap/):

```
$ oc create -f src/main/kube/configmap.yml
```

Create the [Secret](https://kubernetes.io/docs/concepts/configuration/secret/):

```
$ oc create -f src/main/kube/secret.yml
```

Add the [Secret](https://kubernetes.io/docs/concepts/configuration/secret/) to the [ServiceAccount](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/) created earlier:

```
$ oc secrets add sa/camel-xml-swagger-contract-first-sa secret/camel-xml-swagger-contract-first-secret
```

Add the view role to the [ServiceAccount](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/):

```
$ oc policy add-role-to-user view system:serviceaccount:demo-fuse:camel-xml-swagger-contract-first-sa
```

The example can be built and run on OpenShift using the following goals:

```
$ mvn clean install fabric8:resource fabric8:build fabric8:deploy
```

## Testing the code

Use the same tools/instructions for testing standalone (above). Make sure to change the host/port to point to the exposed OpenShift Route.
