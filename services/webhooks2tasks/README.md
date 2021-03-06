# amazeeio-webhooks2tasks

This service is called 'webhooks2tasks', is part of the amazee.io lagoon deployment system and is responsible for converting webhooks into actual tasks that should be executed.

It does the following:
1. read message from a rabbitmq queue called `amazeeio-webhooks`
2. connect to the amazeeio api and load the SiteGroup information for the GitURL in the message (if sitegroup cannot be resolved, logs to amazeeio-logs)
3. analyzing the message and calls a specific handler for the webhooktype and the event name (like githubPullRequestClosed)
4. the handler will then create a task in the correct rabbitmq task queue. (In our example, closed pull requests need to remove openshift resources, so it creates a task in `amazeeio-tasks:remove-openshift-resources`)
5. If no handler is defined for the webhook type or the event, it will log that to `amazeeio-logs`

It uses https://github.com/benbria/node-amqp-connection-manager for connecting to rabbitmq, so it can handle situations were rabbitmq is not reachable and still receive webhooks, process them and keep them in memory. As soon as rabbitmq is rechable again, it will send the messages there.

## Hosting

Fully developed in Docker and hosted on amazee.io Openshift, see the `.openshift` folder. Deployed via Jenkinsfile.

Uses `amazeeio/centos7-node:node6` as base image.

## Development

Can be used with a local nodejs and connect to a rabbitmq of your choice.

        yarn install
        RABBITMQ_HOST=guest:guest@rabbitmqhost yarn run start

Or via the existing docker-compose.yml

        docker-compose up -d
