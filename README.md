# Turbonomic Integrations Reporting Docker Image
This is a base docker image for reporting solutions created by the Turbonomic Integration team, and deployed in a Turbo 7 Kubernetes environment.

# Contents
* Upstream Content of the [turbointegrations/base:1-alpine](https://github.com/turbonomic-integrations/docker-base) image.
* Public Python Modules
  * pymsql
  * numpy
  * lxml
  * vmtreport

# Usage
This image will typically be extended in downstream projects which include bespoke reporting code.

However, the image does have an entrypoint which will run [vmtreport](https://pypi.org/project/vmtreport/) with a config file mapped to `/opt/turbonomic/report.conf` in the container.

An example, using bind mounts in docker might be;

`$ docker run --rm -it -v /path/to/report.conf:/opt/turbonomic/report.conf turbointegrations/reporting:latest`

An example, using a ConfigMap in kubernetes might be;

`cronjob.yml`
```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: reportTest
spec:
  schedule: "* * * * *"
  concurrencyPolicy: Forbid
  suspend: false
  jobTemplate:
    metadata:
      generateName: reportTest
      labels:
        environment: prod
        team: integration
        app: reportTest
    spec:
      template:
        metadata:
          labels:
            environment: prod
            team: integration
            app: reportTest
        spec:
          containers:
          - image: turbointegrations/reporting:latest
            imagePullPolicy: IfNotPresent
            name: reportTest
            volumeMounts:
            - mountPath: /opt/turbonomic
              name: reportConf
          volumes:
            - name: reportConf
              configMap:
                name: reportConf
          restartPolicy: Never          
```

Create a ConfigMap, given an existing file named `report.conf`, and execute the job;

```
$ kubectl create configmap reportConf --from-file=report.conf
$ kubectl apply -f cronjob.yml
$ kubectl create job --from=cronjob/reportTest reportTest
```

## Configuration

Please see the documentation of [vmtreport](https://pypi.org/project/vmtreport/) for details on how to create a configuration file.

# Build

There are three flavors of this image, to match the three flavors of the [turbointegrations/base:1-alpine](https://github.com/turbonomic-integrations/docker-base) image.

1. turbointegrations/reporting:`<version>`-alpine
2. turbointegrations/reporting:`<version>`-slim-buster
3. turbointegrations/reporting:`<version>`-rhel

## Automated build

The Jenkins workflow will be triggered nightly to build new images. A new image, with an automatically incremented semantic version number will be created if any changes are detected.

The Jenkins workflow will;
* Check out all branches of the repository
* Build all three flavors of the image, per the instructions in the `Jenkinsfile`
* If there are changes to the image or encapsulated libraries
  * Push all three flavors of the image to the docker hub repository.
  * Tag the repository with the semantic version number of the build.
