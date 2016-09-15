# TEST DRIVE TOOLING SETUP

## Requirements

- clone this repository, next steps consider that the working directory is that clone
- install recent docker (>= 1.11) and docker-compose (>= 1.7)

## User to migrate selection

You have to generate the mailboxes list to export in a file, that will later be used by the exporter tool.

    docker run -e LOGIN=Administrator -e PASSWORD=passwd linagofab/spews-node-targetselector -g GROUP_TO_EXPORT -u ldap://AD_SERVER -d DC=my,DC=domain,DC=org > targets.csv

## The exporter

The first thing is to adapt the "exporter.config" file. Set the admin's credentials and replace the EXCHANGE_SERVER by the exchange server ip address.

Then, you can begin to fetch data from exchange. Start the container with the following command:

    docker-compose up exporter

You should see some lines printed on the console, the export has started, and it can take a while!

## Save migration data

Once the export has finished, you can make a backup of the generated "rabbitmq-data" folder.
Such backup can be useful in case of failure during the import in OBM.

## The importer

Find the OBM domain UUID to target, you can search it in the database or request PAPI with a curl command like:

    curl -u admin0@global.virt:admin http://OBM_TARGET/provisioning/v1/domains/

To start the migration run:

    PAPI_URL=https://OBM_TARGET/provisioning/v1/ DOMAIN_UUID=46dedaf6-a0be-ce01-8413-31a6fcac304b docker-compose up importer

A "importer-log" folder will be created automatically. Regular logs will be produced in "importer-log/import.log". If any error happen on PAPI for a batch, all items of this batch are logged in "import-log/errors.log", and related messages are send back to RabbitMQ to retry the import later. So at the end, it can remain some messages in the queues.

To have more logs, you can change the log level to debug in the docker-compose.yml file, so  LOG_LEVEL=debug
