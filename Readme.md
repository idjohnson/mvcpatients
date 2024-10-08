

# Testing

You can build locally
```
$ docker build -t mvcpatients:0.5 .
```

And run passing in your settings
```
$ docker run -e DB_HOST=12.34.56.78 -e DB_NAME=patientsdb -e DB_USER=patientuser -e DB_PASSWORD=SuperSecretPassword1234 -p 8888:8080 mvcpatients:0.5
```

# Kubernetes

You can use the manifest, providing you updated the values

```
$ kubectl create ns patientsmvc
namespace/patientsmvc created

$ kubectl apply -f ./Deployment/manifest.yaml -n patientsmvc
secret/db-secret created
deployment.apps/mvcpatients-deployment created
service/mvcpatients-service created
```

Or Helm if you prefer
```
$ helm install mypatientmvc --create-namespace -n patientsmvc --set env.dbHost=12.34.56.78 --set env.dbName=patientsdb --set env.dbUser=patientuser --set env.dbPassword='myBigComplicatedPasswordHere' ./Deployment/chart/
NAME: mypatientmvc
LAST DEPLOYED: Sat Aug 10 12:25:25 2024
NAMESPACE: patientsmvc
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

# Database

This assumes a database exists with a table that can be queried and populated by the user.

Here is the SQL that would create such a table
```
      CREATE TABLE "Patient" (
          "Id" integer GENERATED BY DEFAULT AS IDENTITY,
          "FirstName" text NOT NULL,
          "LastName" text NOT NULL,
          "DateOfBirth" timestamp with time zone NOT NULL,
          "SocialSecurityNumber" text NOT NULL,
          "CreatedDate" timestamp with time zone NOT NULL,
          CONSTRAINT "PK_Patient" PRIMARY KEY ("Id")
      );
```

An example output:
```
psql (12.19 (Ubuntu 12.19-0ubuntu0.20.04.1), server 15.7)
WARNING: psql major version 12, server major version 15.
         Some psql features might not work.
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

defaultdb=> CREATE TABLE "Patient" (
   "Id" integer defaultdb(>           "Id" integer GENERATED BY DEFAULT AS IDENTITY,
defaultdb(>           "FirstName" text NOT NULL,
defaultdb(>           "LastName" text NOT NULL,
defaultdb(>           "DateOfBirth" timestamp with time zone NOT NULL,
defaultdb(>           "SocialSecurityNumber" text NOT NULL,
defaultdb(>           "CreatedDate" timestamp with time zone NOT NULL,
defaultdb(>           CONSTRAINT "PK_Patient" PRIMARY KEY ("Id")
defaultdb(>       );
CREATE TABLE
defaultdb=> \q
```

This can also be done locally with a populated appsettings.json and the migrate command
```
$ dotnet ef migrations add InitialCreate
$ dotnet ef database update
```

# OTLP version

we can install with New Relic OTLP

```
$ helm upgrade mypatientmvc -n patientsmvc --set env.dbHost=isaac-MacBookAir --set env.dbName=patientsdb --set env.dbPort=5432 --set env.dbUser=patientsuser --set env.dbPassword=patientPassword1  --set image.repository=idjohnson/freshmvcapp --set image.tag=1.3  --set imagePullSecrets[0].name=myharborreg --set otel.headers="api-key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxNRAL" --set otel.endpoint="https://otlp.nr-data.net:4318/" ./Deployment/chart/
Release "mypatientmvc" has been upgraded. Happy Helming!
NAME: mypatientmvc
LAST DEPLOYED: Sun Aug 11 19:47:53 2024
NAMESPACE: patientsmvc
STATUS: deployed
REVISION: 5
TEST SUITE: None
```

or a local collector
```
$ helm upgrade mypatientmvc -n patientsmvc --set env.dbHost=isaac-MacBookAir --set env.dbName=patientsdb --set env.dbPort=5432 --set env.dbUser=patientsuser --set env.dbPassword=patientPassword1  --set image.repository=idjohnson/freshmvcapp --set image.tag=1.3  --set imagePullSecrets[0].name=myharborreg --set otel.headers="" --set otel.endpoint="https://otelcollector.opentelementry.svc.cluster.local:4318/" ./Deployment/chart/
Release "mypatientmvc" has been upgraded. Happy Helming!
NAME: mypatientmvc
LAST DEPLOYED: Sun Aug 11 19:47:53 2024
NAMESPACE: patientsmvc
STATUS: deployed
REVISION: 5
TEST SUITE: None
```

Or just dump to a localhost to stop sending metrics, traces and logs
```
$ helm upgrade mypatientmvc -n patientsmvc --set env.dbHost=isaac-MacBookAir --set env.dbName=patientsdb --set env.dbPort=5432 --set env.dbUser=patientsuser --set env.dbPassword=patientPassword1  --set image.repository=idjohnson/freshmvcapp --set image.tag=1.3  --set imagePullSecrets[0].name=myharborreg --set otel.headers="" --set otel.endpoint="http://localhost:4318/" ./Deployment/chart/
```

I can save the changes thus far
```
builder@DESKTOP-QADGF36:~/Workspaces/MvcPatients$ git status
On branch newrelic
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   Deployment/chart/Chart.yaml
        modified:   Deployment/chart/templates/deployment.yaml
        modified:   Deployment/chart/values.yaml
        modified:   Dockerfile
        modified:   MvcPatients.csproj
        modified:   Program.cs
        modified:   Readme.md
        modified:   Views/Home/Index.cshtml
        modified:   Views/Shared/_Layout.cshtml

no changes added to commit (use "git add" and/or "git commit -a")
builder@DESKTOP-QADGF36:~/Workspaces/MvcPatients$ git add -A
builder@DESKTOP-QADGF36:~/Workspaces/MvcPatients$ git commit -m "Updates"
[newrelic 9405dd1] Updates
 9 files changed, 125 insertions(+), 24 deletions(-)
```