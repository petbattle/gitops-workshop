## 🔥🦄 Ubiquitous Journey

At Red Hat Open Innovation Labs, we have automated the bootstrap of Labs Residency CICD tooling to accelerate setup and onboarding. The code repository is called **Ubiquitous Journey** (🔥🦄). We have created a lite version of Ubiquitous Journey here and we will explore this repository and set up our technical foundation using it.

This repo is available on the Red Hat Labs GitHub organization – <span style="color:blue;">https://github.com/rht-labs/ubiquitous-journey.</span> Ubiquitous Journey allows us to plumb all of the pieces together in a developer friendly manner.

- **Extensible** - Our codebase is a *tool box* of code that we can evolve and easily extended to support new tools and methodologies.
- **Traceable** - We can easily see where changes have occurred and most importantly trace exactly what git tag/commit is in which environment.
- **Discoverable** - By making the source code easy to follow, with supporting and inline documentation, new team members can easily discover how application are built, tested and deployed.
- **Auditable** - Git logs and history are the single source of truth for building our software. We can create compliance reports and easily enhance the toolset to support more advanced techniques such as code signing and attestations for all our pipeline steps if needed.
- **Reusable** - Many parts of CICD are reusable. A good example are the reusable pipelines and tasks. Its not only the code however, solid foundational practices such as build once, tag and promote code through a lifecycle can be codified.
- **Flexible** - Product teams often want to use both standard tools and be able to experiment with new ones. The *tool box* mentality helps a lot, so as a team you can work with the tools you are familiar with. We will see this in action with Jenkins and Tekton.

All of these traits lead to one outcome - the ability to build and release quality code into multiple environments whenever we need to.

### Verify if GitLab is Ready for GitOps 🐙
> For this exercise, a Git repository to store our configuration is already set up under your username.

1. Log into GitLab with your credentials to view `tech-exercise` repository. 

    ```bash
    https://<GIT_SERVER>
    ```


### Add ArgoCD Webhook from GitLab
> ArgoCD has a cycle time of about 3ish mins - this is too slow for us, so we can make ArgoCD sync our changes AS SOON AS things hit the git repo.

1. Let's add a webhook to connect ArgoCD to our `ubiquitous-journey` project. Get ArgoCD URL with following:

    ```bash#test
    echo https://$(oc get route argocd-server --template='{{ .spec.host }}'/api/webhook  -n ${TEAM_NAME}-ci-cd)
    ```

2. Go to `tech-exercise` git repository on GitLab. From left panel, go to `Settings > Integrations` and add the URL you just copied from your terminal to enable the WebHook. Now whenever a change is made in Git, ArgoCD will instantly reconcile and apply the differences between the current state in the cluster and the desired state in git 🪄. Click `Add webhook`.

    ![gitlab-argocd-webhook](images/gitlab-argocd-webhook.png)


### Deploy Ubiquitous Journey 🔥🦄
> In this exercise, we'll create our first namespaces and tooling using a repeatable pattern - GitOps.

1. The Ubiquitous Journey (🔥🦄) is just another Helm Chart with a pretty neat pattern built in to create App of Apps in ArgoCD. Let's get right into it - in the your IDE, Open the `values.yaml` file in the root of the project. This values file is the default ones for the chart and will be applied to all of the instances of this chart we create. The Chart's templates are not like the previous chart we used (`services`, `deployments` & `routes`) but an ArgoCD application definition, just like the one we manually created in the previous exercise when we deployed an app in the UI of ArgoCD. You should see these changes already made for you 🪄

    ```yaml
    source: "https://<GIT_SERVER>/<TEAM_NAME>/tech-exercise.git"
    team: <TEAM_NAME>
    ```

2. Go to `tech-exercise` git repository on GitLab by clicking [here](https://<GIT_SERVER>/<TEAM_NAME>/tech-exercise) 👈

    From left panel, go to `Settings > Integrations` and add the URL you just copied from your terminal to enable the WebHook. Now whenever a change is made in Git, ArgoCD will instantly reconcile and apply the differences between the current state in the cluster and the desired state in git 🪄. Click `Add webhook`.

![gitlab-argocd-webhook](images/gitlab-argocd-webhook.png)

3. In order for ArgoCD to sync the changes from our git repository, we need to provide access to it. We'll deploy a secret to cluster, for now *not done as code* but in the next lab we'll add the secret as code and store it encrypted in Git. In your terminal

    Add the Secret to the cluster:

    ```bash
    export GITLAB_USER=<TEAM_NAME>
    ```

    ```bash
    export GITLAB_PASSWORD=<add-your-gitlab-password-here>
    ```

    ```bash#test
    cat <<EOF | oc apply -n ${TEAM_NAME}-ci-cd -f -
      apiVersion: v1
      data:
        password: "$(echo -n ${GITLAB_PASSWORD} | base64 -w0)"
        username: "$(echo -n ${GITLAB_USER} | base64 -w0)"
      kind: Secret
      type: kubernetes.io/basic-auth
      metadata:
        annotations:
          tekton.dev/git-0: https://${GIT_SERVER}
          sealedsecrets.bitnami.com/managed: "true"
        name: git-auth
EOF
    ```

5. Install the tooling in Ubiquitous Journey (only bootstrap at this stage..). Once the command is run, open the ArgoCD UI to show the resources being created. We've just deployed our first AppOfApps!

    ```bash#test
    cd /projects/tech-exercise
    helm upgrade --install uj --namespace ${TEAM_NAME}-ci-cd .
    echo https://$(oc get route argocd-server --template='{{ .spec.host }}' -n ${TEAM_NAME}-ci-cd)
    ```

    ![argocd-bootstrap-tooling](./images/argocd-bootstrap-tooling.png)

6. As ArgoCD sync's the resources we can see them in the cluster:

    ```bash#test
    oc get projects | grep ${TEAM_NAME}
    ```

    ```bash#test
    oc get pods -n ${TEAM_NAME}-ci-cd
    ```

🪄🪄 Magic! You've now deployed an app of apps to scaffold our tooling and projects in a repeatable and auditable way (via git!). Next up, we'll make extend the Ubiquitous Journey with some tooling 🪄🪄
