# Platform Runbook

This is the runbook for the M3O platform and should act as an operations manual for things like outages etc.  
<!--ts-->
   * [Platform Runbook](#platform-runbook)
   * [Namespaces](#namespaces)
      * [Creating a new namespace](#creating-a-new-namespace)
   * [Users](#users)
      * [Add new user to beta](#add-new-user-to-beta)
      * [User deleted their account](#user-deleted-their-account)
      * [Cancel a user's subscription (paid tier)](#cancel-a-users-subscription-paid-tier)
      * [Cancel a user's account (free tier)](#cancel-a-users-account-free-tier)
   * [Services](#services)
      * [Redeploying services](#redeploying-services)
         * [Micro (Server)](#micro-server)
         * [M3O (Runtime Services)](#m3o-runtime-services)
   * [Getting access](#getting-access)
      * [Scaleway](#scaleway)
      * [Kubernetes cluster](#kubernetes-cluster)
   * [Cockroach](#cockroach)
      * [Backups](#backups)
      * [Restore](#restore)
      * [Connecting](#connecting)
      * [Running the go micro unit tests](#running-the-go-micro-unit-tests)
   * [Testing Platform Resiliency](#testing-platform-resiliency)
      * [Killing all pods](#killing-all-pods)
   * [Regression Testing](#regression-testing)
      * [Manually](#manually)
      * [Prerequisites](#prerequisites)
   * [Setup](#setup)
      * [Create the cluster](#create-the-cluster)
      * [Deploy the infra, certs and services](#deploy-the-infra-certs-and-services)
      * [Update micro to the latest snapshot](#update-micro-to-the-latest-snapshot)
      * [Update DNS](#update-dns)
      * [Remove default accounts](#remove-default-accounts)
      * [Setup Stripe](#setup-stripe)
      * [Set Config](#set-config)
      * [Setup the rules](#setup-the-rules)
      * [Run the M3O Services](#run-the-m3o-services)
      * [Tag owners for namespaces](#tag-owners-for-namespaces)
      * [Set up the Cockroach backups](#set-up-the-cockroach-backups)
   * [Troubleshooting](#troubleshooting)
      * [Alerts about certificate expiry](#alerts-about-certificate-expiry)


<!-- Added by: domwong, at: Tue 10 Nov 2020 17:27:45 GMT -->

<!--te-->
To regenerate the table of contents, use [this tool](https://github.com/ekalinin/github-markdown-toc) and do `gh-md-toc --insert platform.md`

# Namespaces

## Creating a new namespace

A quick way to create a new namespace without a customer account

```
micro auth create account --namespace=foobar --secret=secret john
micro platform createNamespace --name=foobar
micro user namespace set foobar
micro login
```

# Users

## Add new user to beta

Users need to have their email added to the "allow" list of the invite service. This will also send an email to the user. 

```
micro invite create --email="dom@m3o.com"
```

## User deleted their account

If a user managed to delete their account by accident (`micro auth delete account foo`) then we need to recreate it. Note: nothing else should be required, creating the auth account should be enough to relink everything.

Find the user's namespace
```
micro namespaces list --owner=dom@m3o.com 
```

Generate their account. Use a strong password for the secret (e.g. https://strongpasswordgenerator.com/) 
```
micro call auth Auth.Generate '{"id":"<email address>" , "type":"user" , "options":{"namespace": "<namespace>"}, "secret" : "<strong password>" }'
```

Send details to user via DM. Create a one time message in privnote to give them the password (https://privnote.com/). Also, ask the user to change their password once they've successfully logged in https://m3o.dev/faq#changing-your-password

## Cancel a user's subscription (paid tier)
If a user wants us to cancel their paid account we can do the following. First confirm their email address so you can look up their customer ID

```
$ micro customers read --email=foo@bar.com
{
	"customer": {
		"id": "841d621c-794d-4472-b393-8ff73c803def",
		"status": "active",
		"created": "1603980254",
		"email": "foo@bar.com",
		"updated": "1603980326"
	}
}
```

Take the customer ID from the response and cancel the subscription
```
$ micro subscriptions cancel --customerID=<CUSTOMER_ID>
{}
```

This will also delete the customer, namespaces etc.

## Cancel a user's account (free tier)
If a user wants us to cancel their free account we can do the following. First confirm their email address so you can look up their customer ID

```
$ micro customers read --email=foo@bar.com
{
	"customer": {
		"id": "841d621c-794d-4472-b393-8ff73c803def",
		"status": "active",
		"created": "1603980254",
		"email": "foo@bar.com",
		"updated": "1603980326"
	}
}
```

Take the customer ID from the response and delete the customer
```
$ micro customers delete --id=<CUSTOMER_ID>
{}
```

# Services

## Redeploying services

### Micro (Server)

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
kubectl set image deployments micro=ghcr.io/m3o/platform:<INSERT_TAG_NAME_HERE> -l micro=server -n server
```

Where `<INSERT_TAG_NAME_HERE>` is your container image tag.
Get that from here https://github.com/orgs/m3o/packages.

### M3O (Runtime Services)

To update an M3O service, run `micro update [source]`, e.g: 
`micro update github.com/m3o/services/status`

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

## Connecting

Put this in a file:

```
apiVersion: v1
kind: Pod
metadata:
  namespace: default
  name: cockroach-client
spec:
  containers:
  - name: cockroach-client
    image: cockroachdb/cockroach:v20.1.3
    imagePullPolicy: IfNotPresent
    command: [ "/bin/bash", "-c", "--" ]
    args: [ "while true; do sleep 30; done;" ]
    volumeMounts:
    - name: cockroachdb-debug-certs
      mountPath: "/certs"
  volumes:
  - name: cockroachdb-debug-certs
    secret:
      secretName: cockroachdb-debug-certs
      defaultMode: 0600     
```

Then (assuming the path of your file is `~cockroach/yml`)

```
kubectl create -f ~/cockroach.yml
```

```
kubectl exec -it cockroach-client -- ./cockroach sql --certs-dir=/certs --host=cockroachdb-cluster
```

When done with debugging

```
kubectl delete -f ~/cockroach.yml
```

## Running the go micro unit tests

By default the cockroach unit tests are simply skipped if there is no cockroach running. If you are hacking on the cockroach store code and want to run the tests locally, do the following:  

Follow the simple installation instructions here: https://www.cockroachlabs.com/docs/stable/install-cockroachdb-linux.html

Start cockroach by:

```
cockroach start-single-node --insecure
```

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
- Get github.com/micro/services/blog/posts running. Example calls in service readmes if [posts](https://github.com/micro/services/tree/master/blog/posts) and [tags](https://github.com/micro/services/tree/master/blog/tags)
- Do some surface testing to ensure
    - We’re not leaking creds
    - We can’t see any store data that we shouldn’t be able to
    - We can’t call any services we shouldn’t be able to
    - We can’t see any bits of infra (etcd, cockroach, k8s) that we shouldn’t be able to

# Deploying Platform

## Prerequisites

Before deploying the platform you will need:
- Access to Scaleway
- Access to Cloudflare
- The slack API key
- The stripe API key
- The Sendgrid API key
- A Cloudflare API token, with access to m3o.com & m3o.app with Zone.Zone and Zone.DNS permissions.
- The reference of the latest stable micro docker image snapshot.

# Setup

## Create the cluster

Create the cluster in Scaleway, running Kubernetes 1.18.6. The default pool should use the "GP1-S” tier which has 32GB of ram. Do not enable auto-scaling.

Wait for the node pools to become Ready. Whilst you wait, download the Kubeconfig and set it locally `export KUBECONFIG=~/Download/micro-platform.yaml`

## Deploy the infra, certs and services

Go to micro/micro/platform/kubernetes and run `bash install.sh production`. This script will install the certs, infra and all the micro services (e.g. runtime).

Once the script has finished, you need to manually add the cloud flare and slack tokens. Do this by running `kubectl edit secret micro-secrets` and adding the keys 'cloudflare' and 'slack' The values should be base64 encoded.

Wait for all the pods to be “Running”.

## Update micro to the latest snapshot

Update micro to the latest stable docker image snapshot using the following command and then wait for the services to all have the status "Running".

```bash
kubectl set image deployments micro=ghcr.io/m3o/platform:tag -l micro=runtime
```

## Update DNS

Update the DNS records in cloudflare to point to the k8s service Public IP for proxy and api. Once this is done, you should be able to call the service via the API and proxy using the following commands:

```bash
micro env set platform
micro call store Debug.Health
curl https://api.m3o.com/store/Debug/Health
```
## Remove default accounts

Firstly, login as admin:
```bash
micro login --email=admin --password=micro
```

Create yourself an account, noting down the name and secret which is generated.
```bash
micro auth create account [name]
```

Login as your new account
```bash
micro login --email=[name] --password=[password]
```

Delete the admin account: 
```bash
micro auth delete account admin
```

 Verify it worked by listing the accounts. The only one listed should be the one you just created
```bash
micro auth list accounts
```

## Setup Stripe

Create the product and plan in stripe. The response contains an ID, you'll need that for the next command so note it down.
```bash
curl https://api.stripe.com/v1/products \
  -u [strip api key]: \
  -d name="M3O Platform" \
  -d description="M3O Platform Subscription"
```

Then create the plan. Use the product ID from above. This will return you a plan ID which is needed in the next step.
```bash
curl https://api.stripe.com/v1/plans \
  -u [stripe api key]: \
  -d amount=3500 \
  -d currency=usd \
  -d interval=month \
  -d product=[product id, created above]
```

## Set Config

Set the required config. Replace the substituted values with the production API keys etc.

```bash
micro config set micro.alert.slack.token [slack api key]
micro config set micro.alert.slack.enabled true
micro config set micro.payments.stripe.api_key [stripe api key]
micro config set micro.emails.sendgrid.api_key [sendgrid api key]
micro config set micro.emails.email_from "Micro Team <support@m3o.com>";
micro config set micro.emails.enabled true;
micro config set micro.signup.sendgrid.template_id d-240bf196257143569539b3b6b82127c0;
micro config set micro.signup.sendgrid.recovery_template_id d-08c2330ae2824de5b2730e49e298e97e;
micro config set micro.invite.sendgrid.invite_template_id d-2d107482af6d47f8a721315906ada753;
micro config set micro.subscriptions.plan_id [stripe plan id];
micro config set micro.subscriptions.additional_users_price_id [stripe additional users price id];
micro config set micro.signup.email_from "Micro Team <support@m3o.com>";
micro config set micro.status.services "api,auth,broker,config,network,proxy,registry,runtime,status,store,signup,platform,invite,payment,customers,namespaces,subscriptions,emails,alert,billing";
```
 
If the environment should run as a "free" tier with no stripe payments then you should set the following config

```bash
micro config set micro.signup.no_payment true
micro config set micro.signup.message "Finishing signup for %s"
```

TODO: update docs to define exactly which services are required for free vs paid.

Verify the config by calling`“micro config get micro`. This will output the config as JSON.

## Setup the rules

Setup the auth rules to restrict access to the m3o services before we deploy them:
```bash
micro auth create rule --resource="service:alert:*" --priority 1 alert
micro auth create rule --resource="service:status:*" --priority 1 status
micro auth create rule --resource="service:auth:*" --priority 1 auth
micro auth create rule --resource="service:signup:*" --priority 1 signup
micro auth create rule --resource="service:runtime:*" --priority 1 runtime-public
micro auth create rule --resource="service:registry:*" --priority 1 registry-public
micro auth create rule --resource="service:store:*" --priority 1 store-public
micro auth create rule --scope='*' --resource="*:*:*" onlyloggedin
micro auth delete rule default
```

## Run the M3O Services

Run the build service and wait for it to start before running anything else:
```bash
micro run github.com/m3o/services/build
```

Run the M3O services which make up the platform:
```bash
micro run github.com/m3o/services/payments
micro run github.com/m3o/services/alert
micro run github.com/m3o/services/signup
micro run github.com/m3o/services/emails
micro run github.com/m3o/services/status
micro run github.com/m3o/services/invite
micro run github.com/m3o/services/api/client
micro run github.com/m3o/services/platform
micro run github.com/m3o/services/platform
micro run github.com/m3o/services/gitops
micro run github.com/m3o/services/customers
micro run github.com/m3o/services/subscriptions
micro run github.com/m3o/services/namespaces
```

Wait for the services to all be running. This can be checked by running `micro services`

## Tag owners for namespaces
The `default` and `micro` namespaces need to be correctly tagged as being owned by us so add label `owner: micro` to the metadata labels for these namespaces 
```
kubectl edit namespace default 
kubectl edit namespace micro 
```

## Set up the Cockroach backups
See [previous section](#backups)

# Troubleshooting

## Alerts about certificate expiry

If you see regular alerts about in Slack about certificate expiry e.g. `[FIRING:1] CertManagerCertExpirySoon` it might be because cert-manager is having issues renewing the cert. All certs should auto renew so if they don't we have a problem. Check the logs for the `cert-manager` pod

```
stern -n cert-manager certmanager-cert-manager-767dbf6fdb-6tfz2
```

and you might see an error like

```
certmanager-cert-manager-767dbf6fdb-6tfz2 cert-manager E0407 14:20:45.149593       1 controller.go:158] cert-manager/controller/CertificateKeyManager "msg"="re-queuing item  due to error processing" "error"="Operation cannot be fulfilled on certificates.cert-manager.io \"tls-api\": the object has been modified; please apply your changes to the latest version and try again" "key"="server/tls-api" 
```

If this keeps occurring you may have to delete and recreate the certificate object. Save the existing certificate

```
k get certificate -n server tls-api --export -o yaml > cert-tls-api.yaml
```

Then delete it from k8s

```
 k delete certificate -n server tls-api  
```

Then recreate it, which will force cert-manager to try and reissue it

```
k apply -f cert-tls-api.yaml -n server  
```

