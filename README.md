## Table of contents

1. [About](##about)
2. [Setup](##setup)
3. [Credentials](##credentials)


## About

This is an ArgoCD App of Apps deployment represented in helm charts. It deploys RHSSO, Gitea, and Quay. This is the App of App structure: 

    -- applications (Parent App)
    |-- catalogsource (Child App)
    |-- configs (Child App)
    |   |-- gitea-config
    |   |-- quay-config
    |   `-- rhsso-config
    |-- subscriptions (Child App)
    |   |-- gitea
    |   |-- quay
    |   `-- rhsso
    `-- tekton (Child App)
        |-- oc-client-task
        |-- oc-copy-router-ca-task
        |-- create-admin-secret-task
        |-- create-rhsso-gitea-client-task
        |-- create-rhsso-ocp-client-task
        |-- create-rhsso-quay-client-task
        |-- patch-gitea-deployment-ca-task
        `-- create-oidc-in-gitea-task

ArgoCD will sync the helm resources from this repository using the app.yaml located in the root directory. The operator subscriptions ensure the operators get deployed. The *-configs eventualy get picked up by the operators and deploy the corresponding applications(gitea,rhsso, .. ) defined by the custom resources (variables configurable via helm). 

The tekton tasks and pipeline resources are also made available via ArgoCD and once kicked off, they provision the integration between RHSSO and (gitea, quay, etc). 

For configuring the applications to work with each other, we utilize tekton tasks to create a modular approach. The tasks define default variables that are easily customized by the user when they create a pipeline. By having a task for each part, we modularize the automation, thus allowing us to create and add additional pieces of automation - GitOps style. Tasks are easy to test via the form builder in Openshift Pipelines and also tremendously easy to troubleshoot. 

## Disclaimer

Currently the way Openshift GitOps is handled in Openshift, the argocd environment utilizes the ```argocd-cluster-argocd-application-controller service account``` to manage all deployemnts. However, by default the service account only has access to the openshift-gitops namespace. See here: 
https://github.com/argoproj-labs/argocd-operator/issues/200


In order for it to manage other openshift namespaces, you must give it cluster-admin privileges. See here: https://argocd-operator.readthedocs.io/en/latest/install/openshift/#rbac

TLDR; Run this command: 

```
oc adm policy add-cluster-role-to-user cluster-admin -z argocd-cluster-argocd-application-controller -n openshift-gitops
```

The alternative, which is a best practice, is to register your Openshift environment within ArgoCD using an admin account. Instead of using the default kubernetes url: https://kubernetes.default.svc, you can select your cluster url instead. 

## Setup 

Deploy ArgoCD into your cluster via the ArgoCD operator or Openshift GitOps operator.

See [disclaimer](##disclaimer) above.

Apply the ArgoCD App Resource inside the ArgoCD **Create New Application** section.

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: super-cool-app
  #namespace: default
spec:
  destination:
    name: ''
    namespace: dso
    server: 'https://kubernetes.default.svc'
  source:
    path: applications
    repoURL: 'https://github.com/tonykhbo/dso'
    targetRevision: app-of-apps-tobo
    helm:
      releaseName: super-cool-app
      valueFiles:
        - values.yaml
      values: |
        catalogsource:
          new: value
        subscriptions:
          more: values
        config:
          yo: wassup
  project: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

```

For custom configurations, refer to the values.yaml file available for the Parent App or Child Apps. 

To configure the integration between the Child Apps, see README in the [tekton directory](https://github.com/tonykhbo/dso/tree/app-of-apps-tobo/tekton). 

## Credentials

The username/password credentials for the child apps are located in the corresponding namespace (default is dso) > secret. 

