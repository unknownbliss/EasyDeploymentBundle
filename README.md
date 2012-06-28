EasyDeploymentBundle
================

This bundle provide an easy way of deploying Symfony2 applications.

Folder structure
-------------------

This bundle will create in your remote server the next folder structure

    /my/application.com/
        current/ -> /my/application.com/releases/20120101232343/ (this should be your documentroot)
        releases/
            20120101232343/
                app/
                web/
                src/
                # ...
            20121023123245/

How to use
----------

Deploy application to server:

    php app/console easydeploy:deploy server_name

Test application deployment to server:

    php app/console easydeploy:deploy server_name --mode=test

Deploy application to a group of servers:

    php app/console easydeploy:deploy:group group_name

Something went wrong? Rollback to previous version:

    php app/console easydeploy:rollback server_name
    php app/console easydeploy:rollback:group group_name

Things were better before? Rollback to specific version:

    php app/console easydeploy:rollback server_name --version=20120101000000
    php app/console easydeploy:rollback:group group_name --version=20120101000000

Execute a specific task in a server:

    php app/console easydeploy:task server_name --task=task1
    php app/console easydeploy:task:group group_name --task=task1 --task=task2 --onfailure=abort|continue

Clear old releases:

    php app/console easydeploy:clear server_name --keep_last=2
    php app/console easydeploy:clear:group server_name --keep_last=1

Do deployment silently:

    php app/console easydeploy:deploy server_name --silent

Configuration reference
-----------------------

    easy_deployment:

        # this is the global application name
        application_name: myapplication.com

        keyring:
            name:
                public_key: /path/to/file
                private_key: /path/to/file
                passphrase: passphrase

        # here you can define tasks to be executed in your server.
        # the types are defined in the core, you can also define
        # another types of tasks by giving the full class name
        tasks:
            task_1:
                # set the mode this task will run only. If no modes sets
                # it will be executed in all modes. Use this to execute
                # task, for example phpunit, only in test mode
                modes: [live, test]
                type: symfony
                command: cache:warmup
            task_2:
                # this is a special type of task that will be handle internally
                type: doctrine_migrations
            task_3:
                type: shell
                arguments: [./dosomething.sh]
            task_4:
                type: shell
                arguments: [./dosomethingnotreallynecessary.sh]
                # if you are also doing some unimportant task, you can ignore it in case of failure
                # by default, all tasks abort the deploy and triggers the rollback when they fail
                on_failure: continue
            task_5:
                # your task must implement TaskInterface
                class: My\Custom\Task\Type\Class
                arguments: [something]

        servers:
            server:
                host: host
                username: username
                password: password
                # use this key, ignores password settings if any
                key: name
                paths:
                    # this is the live folder, the document root
                    live: /path/to/live/folder
                    # this is the test folder, when you do a no dry-run test, tests files
                    # will be keep here, good for doing some testing against production data
                    test: /path/to/test/folder
                rules: [rule, rule]
                # this sets the number of success deploys to keep in
                # the temporary folder
                keep_releases: 5
                settings:
                    ssh_port: 22
                    rsync_port: 222
                    php_bin: /path/to/php

            server2:
                inherit: server
                host: host
                username: username

        rules:
            symfony:
                ignore: [rule, rule, rule]


        groups:
            name:
                servers: [server, server2]
                on_abort: abort_all/continue

        settings:
            ssh:
                default_key: name

            rsync:
                delete: true
                follow_symlinks: true

            testing:
                # this wont touch servers at all, if false, test folder will be populated
                dry_run: true

            # entity manager where the deployer will
            entity_manager: name

        # shared files between releases
        shared:
            # files that must be syncronyzed from local but are
            # common in all releases
            local:
                - web/public/*
                - app/DoctrineMigrations/*