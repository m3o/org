# Runbooks

This directory is a repository of runbooks for various parts of the M3O system and should act as an operations manual for things like outages etc.  
Currently a single file but parts will be moved out once they become overly long. To regenerate the table of contents, use [this tool](https://github.com/ekalinin/github-markdown-toc) and do `cat ./README.md | gh-md-toc -`.

# Table of Contents

   * [Users](#users)
   * [Add new user to beta](#add-new-user-to-beta)
   * [Services](#services)
      * [Redeploying services](#redeploying-services)
   * [Getting access](#getting-access)
      * [Scaleway](#scaleway)
      * [Kubernetes cluster](#kubernetes-cluster)
   * [Cockroach](#cockroach)
      * [Backups](#backups)
      * [Restore](#restore)
   * [Testing Platform Resiliency](#testing-platform-resiliency)
      * [Killing all pods](#killing-all-pods)
   * [Regression Testing](#regression-testing)
      * [Manually](#manually)


# Users

# Add new user to beta

Users need to have their email added to the "allow" list of the invite service.

```
micro invite create --email="me@domwong.com"
```

# Services

## Redeploying services

Assuming you have access to [Kubernetes](#-kubernetes-cluster).

You can redeploy core services with zero downtime by setting the image. Full info here: https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-interactive/

Short guide here:

```
$ kubectl get deployments
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
micro-api        1/1     1            1           3d22h
micro-auth       1/1     1            1           4d13h
micro-broker     1/1     1            1           4d13h
micro-config     1/1     1            1           4d13h
micro-debug      1/1     1            1           4d13h
micro-network    1/1     1            1           4d
micro-proxy      1/1     1            1           3d22h
micro-registry   1/1     1            1           4d13h
micro-router     1/1     1            1           2d23h
micro-runtime    1/1     1            1           4d13h
micro-store      1/1     1            1           4d13h
nats-operator    1/1     1            1           4d14h
```

```
kubectl set image deployments/micro-runtime runtime=micro/micro:tag
```

Please note `tag` must be changed to your specific tag.

# Getting access

## Scaleway

You should ask on Slack to be invited to the Scaleway organisation.

## Kubernetes cluster

Assuming you have [Scaleway](-scaleway) access, go to scaleway, choose the relevant kapsule and grep for "Get kubeconfig".
Then either an ad hoc envar (eg. `KUBECONFIG=/path/to/kubeconfig kubectl get pods`) can be used when interacting with kubectl or the same envar can be placed
in the `bashrc` file (eg. `export KUBECONFIG=/path/to/kubeconfig`). PS: don't forget to source your `bashrc`.

# Cockroach

## Backups
Our Kubernetes cluster has a cron job which will dump Cockroach and push to S3 every hour. You can find the code for this [here](https://github.com/m3o/services/tree/master/cockroachdb/backup).

## Restore
If you need to restore Cockroach from a backup you can do this by running the Kubernetes pod defined [here](https://github.com/m3o/services/tree/master/cockroachdb/restore) on a fresh Cockroach cluster. When the pod runs, it will download the latest dump file from S3 and apply to the cluster. Once complete, don't forget to delete the pod.
1. `kubectl apply -f cockroach-restore.yaml`
2. `kubectl delete po cockroach-restore`

# Testing Platform Resiliency

## Killing all pods

Please note the `sleep 120` part in the codes below.

Kill core pods:

```
kubectl get pods -n default | grep micro | awk '{print $1}' | xargs -I % sh -c '{ kubectl delete pods  %; sleep 120; }'
```

Kill non core pods in the micro namespace to redeploy invite and other services:

```
kubectl get pods -n micro | awk '{print $1}' | grep -v NAME | xargs -I % sh -c '{ kubectl delete pods % -n micro; sleep 120; }'
```

# Regression Testing

## Manually

- Do full signup flow
- Get github.com/micro/services/blog/posts running
- Do some surface testing to ensure
    - We’re not leaking creds
    - We can’t see any store data that we shouldn’t be able to
    - We can’t call any services we shouldn’t be able to
    - We can’t see any bits of infra (etcd, cockroach, k8s) that we shouldn’t be able to