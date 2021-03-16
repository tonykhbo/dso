To test a task in Openshift, you can navigate to the Pipelines tab of the Dashboard and create a pipeline. Add an individual task and select it. It will display a form with default variables that you can modify to fit your own environment.

When we configure our pipeline, we will define the tasks in this directory in the following order:


        1|-- oc-copy-router-ca-task
            2|-- patch-gitea-deployment-ca-task
        1|-- create-admin-secret-task
                3|-- create-rhsso-ocp-client-task
                3|-- create-rhsso-quay-client-task
                3|-- create-rhsso-gitea-client-task
                    4|-- create-oidc-in-gitea-task

After creating the pipeline and editting your variables to suit your environment, you can start the pipeline. 

Before you can login to Gitea to test out the OIDC integration, you must first create a user inside RHSSO. Navigate to the RHSSO route, login as admin using the credentials defined in the deployment namespace > secret. 

After creating and verifying the user credentials work in RHSSO, you can test out the login in Gitea. 

Quay RHSSO integration tasks coming eventually..