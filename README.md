# Flux Compose Demo

This repo provides a demonstration of using a Compose file to define an application running in Kubernetes and deployed using Flux. It is making use of my [compose-deployer Helm chart](https://github.com/mikesir87/helm-charts/charts/compose-deployer), but it can easily be swapped out with any other tooling. 

The general idea is to have a pipeline that converts the manifests and commits those back to the repo where Flux can pick them up. In this example, we are using GitHub Actions, which can easily be swapped out with whatever CI system you might be using.

## Running it Yourself

If you want to exercise the full demo, you will want to do the following:

1. Fork this repo (which will let you make changes to the files)
1. In the `flux/manifests.yaml` file, adjust the `url` for the GitRepository to point to your forked repo
1. Clone the project
1. If you need Traefik/Flux to be installed, you can do so using the manifests in the `/pre-reqs` directory:

    ```shell
    kubectl apply -k ./pre-reqs
    ```

1. Now, we are going to define the `GitRepository` and `Kustomization` objects that provide config to Flux to watch and apply the manifests in our repo. Apply the manifests in the `flux` directory:

    ```shell
    kubectl apply -f ./flux
    ```

1. After a moment, you should see the voting app up and running. It might take a minute to pull all of the images.

    ```shell
    > kubectl get pods
    db-677f6fd579-8vcmq       1/1     Running   0          2m
    redis-6c7f9657c-vmq75     1/1     Running   0          2m
    result-66b4856489-7swvq   1/1     Running   0          2m
    vote-65d4d9594f-c7d27     1/1     Running   0          2m
    vote-65d4d9594f-ldd26     1/1     Running   0          2m
    worker-7497c578cb-4g6xf   1/1     Running   0          2m
    worker-7497c578cb-9ztgj   1/1     Running   1          2m
    ```

    You should be able to open [http://vote.localhost](http://vote.localhost) and see the voting app. Opening [http://results.localhost](http://results.localhost) should give you the results page (Chrome auto-resolves *.localhost to localhost, so hopefully it works by default for you).

 1. Make a change to the `docker-compose.yml` file. A good idea might be is to define the `OPTION_A` and `OPTION_B` environment variables on the voting app to change what you're voting for. If you push the file, you should see a pipeline get triggered and the app be deployed on your local machine within a moment or two.


### Cleaning up

 When you're ready to tear everything down, simply remove the flux config, which will then remove all of the apps:

 ```shell
kubectl delete -f ./flux
 ```

 And if you want to remove the Traefik ingress and Flux components, you can do so using this command:

 ```shell
kubectl delete -k ./pre-reqs
 ```
 