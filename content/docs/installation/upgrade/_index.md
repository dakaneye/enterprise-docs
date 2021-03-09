---
title: "Upgrading Anchore Enterprise"
linkTitle: "Upgrade"
weight: 10
---

Upgrading from one version of Anchore Enterprise to another is normally handled seamlessly by the Helm chart or 
trial docker-compose configuration files that are provided along with each release. Those follow the general methods from this guide. 
See [Specific Instructions](#specific-versions) section for special instructions related to specific versions.

Anchore is distributed as a docker image, which is comprised of smaller micro-services that can be deployed in a single container or scaled out to handle load.

The latest version of the Anchore image will be tagged with both the latest tag and a version number. For example **latest** and **v0.7.1**.

To retrieve the version of a running instance of Anchore, the system status command can be run.

```
# anchore-cli system status
...
...
...

Engine DB Version: 0.0.13
Engine Code Version: 0.7.0
```

In this example the Anchore version is 0.7.0 and the database schema is version 0.0.13.  In cases where the database schema is changed between releases, Anchore will upgrade the database schema at launch.

### Pre-upgrade Procedure

Prior to upgrading Anchore, we highly recommend performing a database backup/snapshot by stopping your Anchore installation, and backing up the database in its entirely.  There is no automatic downgrade capability, thus the only way to downgrade after an upgrade (whether it succeeds or fails) is to restore your database contents to a state from a prior version of Anchore, and explicitly run the compatible version of Anchore against the corresponding database contents. 

Whether or not you wish to have the ability to downgrade, we recommend backing up your Anchore database prior to upgrading the software as a best practice.

### Upgrade Procedure (for deployments using Helm)

For the latest upgrade instructions using the Helm chart, please refer to the official Anchore Helm Chart documentation

- [Anchore Helm Chart](https://github.com/anchore/anchore-charts/blob/master/stable/anchore-engine)


### Upgrade Procedure (example with docker-compose)

1. Stop all running instances of Anchore
```
# docker-compose down
```

2. Make a copy of your original docker-compose.yaml file as backup
```
# cp docker-compose.yaml docker.compose.yaml.backup
```

3. Download the latest docker-compose.yaml
```
# curl https://docs.anchore.com/current/docs/quickstart/docker-compose.yaml
```

4. Review the latest docker-compose.yaml and merge any edits/changes from your original docker-compose.yaml.backup to the latest docker-compose.yaml

5. Restart the Anchore containers
```
# docker-compose up -d
```

To monitor the progress of your upgrade, you can watch the docker logs from your catalog container, where you should see some initial output indicating whether or not an upgrade is needed or being performed, followed by the regular Anchore log output.

```
# docker-compose logs -f catalog
```

Once completed, you can review the new state of your Anchore install to verify the new version is running using the regular system status command.

```
# anchore-cli system status
...
...
...

Engine DB Version: 0.0.13
Engine Code Version: 0.7.1
```

### Advanced / Manual Upgrade Procedure

If for any reason the automated upgrade fails, or you would like to perform the upgrade of the anchore database manually, you can use the following (general) procedure.  This should only be done by advanced operators after backing up the anchore database, ensuring that the anchore database is up and running, and that all running anchore components are stopped.

- Install the desired Anchore container manually
- Run the Anchore container but override the entrypoint to run an interactive shell instead of the default 'anchore-manager service start' entrypoint command
- Manually execute the database upgrade command, using the appropriate db_connect string.  For example, if using Postgres, the db_connect string will look like `postgresql://$ANCHORE_DB_HOST/$ANCHORE_DB_NAME?user=$ANCHORE_DB_USER&password=$ANCHORE_DB_PASSWORD`

```
# anchore-manager db --db-connect "postgresql://$ANCHORE_DB_HOST/$ANCHORE_DB_NAME?user=$ANCHORE_DB_USER&password=$ANCHORE_DB_PASSWORD" upgrade
[MainThread] [anchore_manager.cli.utils/connect_database()] [INFO] DB params: {"db_connect_args": {"timeout": 86400, "ssl": false}, "db_pool_size": 30, "db_pool_max_overflow": 100}
[MainThread] [anchore_manager.cli.utils/connect_database()] [INFO] DB connection configured: True
[MainThread] [anchore_manager.cli.utils/connect_database()] [INFO] DB attempting to connect...
[MainThread] [anchore_manager.cli.utils/connect_database()] [INFO] DB connected: True
...
...
```
- The output will indicate whether or not a DB upgrade is needed, prompt for confirmation if it is, and will display upgrade progress output before completing.


## Specific Version Upgrades {#specific-versions}
---
This section is intended as a guide for any special instructions and information related to upgrading to specific versions of Enterprise.

### Upgrading Enterprise 2.3 to 2.4+

The major change between 2.3.x and 2.4+ is that the Helm chart repo has moved from helm/stable to charts.anchore.io. To upgrade from a previous version of the chart, you will need to delete your previous deployment and redeploy Anchore Enterprise using the chart from the Anchore Charts repository.

Detailed instructions are available in the [anchore-charts git repo](https://github.com/anchore/anchore-charts/tree/master/stable/anchore-engine#migrating-to-the-new-anchore-charts-repository).

---

### Upgrading Enterprise 2.2 (Image anchore/enterprise:v0.6.1) to 2.3.0


**NOTE: This upgrade involves a DB schema update from 0.0.12 -> 0.0.13 for the Engine-based components. The upgrade process migrates some data from the 
old centos feed using RHSA data to the new rhel feed using CVE data and may take 10+ minutes since image results are updated and the feeds are configured 
for the new data. The specific time will depend on your deployment resources and the images in your database. More RHEL/CentOS images will take longer 
to process.**

As with previous upgrades, the new version of the Helm chart coinciding with the release includes all necessary updates to the configuration to support 
Enterprise 2.3.0. The specific changes it includes are:

* Uses a specific post-upgrade hook for performing the DB upgrade rather than doing it during bootstrap of any one service. This ensures a cleaner 
upgrade experience with specific upgrade logs available in the upgrade job's container logs.


* All non-UI services are started from the docker.io/anchore/enterprise:v2.3.0 image instead of a mix of engine and enterprise containers. This is 
required for supporting new enterprise features like NuGet and Windows analysis.

* Updates to the entry points of the Enterprise containers for the API, Catalog, Policy Engine, SimpleQueue, and Analyzer services in support of 
moving all services to be started as Enterprise services with version 2.3.0 instead of a mix of Enterprise and Engine services. The new container 
start command is `anchore-enterprise-manager` for all non-UI services.

  * The command executed to start the services changes from:


        anchore-manager service start <apiext|catalog|policy_engine|simplequeue|analyzer>


    to:


        anchore-enterprise-manager service start <apiext|catalog|policy_engine|simplequeue|analyzer>


    in services containers that were previously Engine services.

* All Anchore service containers must have the license.yaml file available at `/license.yaml` (this is also handled in the chart and docker-compose.yaml 
updates) to enable the enterprise-versions of the engine services to start.


The upgrade process itself remains the same:

1. Shut down the previous deployment

1. Update the chart to the newest version (usually requires a `helm repo update` to fetch the newest version). Then, start the new deployment. The DB 
will upgrade automatically as part of the deployment.

### Additional Steps for the RHSA -> CVE Change in 2.3.0

The upgrade will run and complete by itself. Due to a feeds change in 2.3.0 to migrate to CVE matches instead of RHSA matches for RHEL/CentOS images, 
there will be vulnerability matches for both types after the new feed groups sync. You can leave these matches or remove them by flushing the old data
manually. Existing RHEL/CentOS/UBI images analyzed before the upgrade will have both RHSA and CVE matches. New images analyzed after upgrade will have 
only CVE matches against the new data.

**For more information on the upgrade process and how to flush the old RHSA matches see: [RHSA to CVE Migration]({{< ref "/docs/releasenotes/2.3.0/centos_to_rhel_upgrade">}})**


#### Upgrading with Helm Chart

The new version of the Helm chart for Anchore Engine/Enterprise has a few significant changes:

1. Explicit db upgrade process executed via 2 post-upgrade hook jobs. These will run and execute the db schema upgrade while the new versions of the 
service pods wait for the db schema to be updated.

Doing the upgrade:

1. Pull latest chart: `helm repo update`

1. Review any custom values.yaml configuration and compare to the defaults in the chart. Either use 

    `helm pull anchore/anchore-engine`
 
    to view the updated chart, or view on [Helm Charts Github repository](https://github.com/anchore/anchore-charts/blob/master/stable/anchore-engine)

1. Upgrade your deployment: `helm upgrade <your release name, e..g anchore> anchore/anchore-engine <-f custom_values.yaml>`

1. The command will wait while the upgrade jobs complete before returning a success. This may take some time as the database upgrade involves some data migrations.


#### Upgrading with Docker Compose

We do not recommend just changing the image reference in an existing Docker Compose file unless you are sure the configuration is valid for the new release. The expected
method if using an Anchore-provided docker-compose.yaml file from the Quickstart guide, is to replace the previous docker-compose.yaml with the new one, as below.

From the directory in your system where you have the previous docker-compose.yaml file:


1. Download the new docker-compose.yaml file
    ```
    curl https://docs.anchore.com/current/docs/quickstart/docker-compose.yaml > docker-compose-new.yaml
    ```

1. Create backup of old compose file:
    ```
    cp docker-compose.yaml docker-compose.yaml.old
    ```

1. Shut down the deployment but leave volumes in place:
    ```
    docker-compose down
    ```

1. Substitute the new docker-compose.yaml file
    ```
    cp docker-compose-new.yaml docker-compose.yaml
    ```

1. Start the deployment back up:
    ```
    docker-compose up -d
    ```


### Additional Steps for the RHSA -> CVE Change in 2.3.0

The upgrade will run and complete by itself. Due to a feeds change in 2.3.0 to migrate to CVE matches instead of RHSA matches for RHEL/CentOS images, there will be vulnerability
matches for both types after the new feed groups sync. You can leave these matches or remove them by flushing the old data manually. Existing RHEL/CentOS/UBI images analyzed before
the upgrade will have both RHSA and CVE matches. New images analyzed after upgrade will have only CVE matches against the new data.

For more information on the upgrade process and how to flush the old RHSA matches see: [RHSA to CVE Migration]({{< ref "/docs/releasenotes/2.3.0/centos_to_rhel_upgrade">}})

#### Enterprise 2.3.0 Manual / Advanced Upgrade

If you are manually deploying or using a method other than our provided installation tools, it is recommended that you inspect the latest Helm chart and/or the latest docker-compose.yaml
 to review the addition of the reporting service, reporting service configuration environment variables and section in the enterprise service config.yaml, and the addition of the new 
 required configuration options to your existing UI config-ui.yaml configuration file.
