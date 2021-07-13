# DCP Demo

This demo uses Emojivoto (https://github.com/BuoyantIO/emojivoto), but has some additional modifications that were necessary in order to run with a more recent gRPC go library and on MacOS.

The Cluster is set up such that there is an `emojivoto-dev` namespace with the emojivoto application deployed, and an `emojivoto` namespace that has the production version that is managed in Argo Rollouts.



## Telepresence Hard Reset

In order to do a hard-redeploy of the emojivoto-dev namespace (if you're running into intercept issues), run the following:

1. `telepresence quit`
2. `kubectl delete deploy -n emojivoto-dev --all`    
3. `kubectl delete svc -n emojivoto-dev --all`
4. (from dev-rel-01 directory) `git checkout dev`
5. `cd yaml/emojivoto`
6. `kubectl apply -f emoji-svc`                  
7. `kubectl apply -f emoji-voting`    
8. `kubectl apply -f emoji-web`          
9. `cd .. && git checkout main`  

## Service Catalog Notes

On the Develpment card: Emoji-Web [web-svc -n emojivoto] annotations and descriptions  

```yaml
metadata:
  annotations:
    a8r.io/description: gRPC API for voting and leaderboard. Lanaguage = Go
    a8r.io/owner: Daniel Bryant
    ### Opens slack and shows Daniel Bryant's slack profile
    a8r.io/chat: https://datawire.slack.com/team/U8SPRLSQK
    ### Issues page on application repository
    a8r.io/bugs: https://github.com/aes-t2/dev-rel-01/issues
    ### Goes to Ambassador Labs docs: Log levels and debugging
    a8r.io/logs: https://www.getambassador.io/docs/edge-stack/latest/topics/running/running/#log-levels-and-debugging
    ### Goes to the Ambassdor Labs docs homepage
    a8r.io/documentation: https://www.getambassador.io/docs/latest/
    ### Emojivoto repository
    a8r.io/repository: https://github.com/aes-t2/dev-rel-01.git  
    ### Ambassador Labs Support Portal Login Page
    a8r.io/support: https://support.datawire.io/hc/en-us
    ### Ambassador docs debugging page
    a8r.io/runbook: https://www.getambassador.io/docs/latest/topics/running/debugging/
    ### REQUIRES LOGIN: Datadog incident panel (Only shows demo screen)
    a8r.io/incidents: https://app.datadoghq.com/incidents/settings
    ### Jaeger Trace Boutique App.  Frontend to Shipping & Product Catalog
    a8r.io/uptime: https://monitoring.dr.mturner.k736.net/jaeger/trace/98ab4d0fa605c2f391340c299bfe2ef0
    ### Login before presenting. In most cases login is cached and not required each time. Username = admin, PW = admin
    a8r.io/performance: https://monitoring.dr.mturner.k736.net/grafana/d/R_NuxHVWk/ambassador-dashboard?refresh=1m&orgId=1
    a8r.io/dependencies: voting-svc.emojivoto-dev, emoji-svc.emojivoto-dev
```

On the Production card: Emoji-Web [web-svc -n emojivoto] annotations and descriptions  

```yaml
metadata:
  annotations:
    a8r.io/description: gRPC API for voting and leaderboard. Lanaguage = Go
    a8r.io/owner: Daniel Bryant
    ### Opens slack and shows Daniel Bryant's slack profile
    a8r.io/chat: https://datawire.slack.com/team/U8SPRLSQK
    ### Issues page on application repository
    a8r.io/bugs: https://github.com/aes-t2/dev-rel-01/issues
    ### Goes to Ambassador Labs docs: Log levels and debugging
    a8r.io/logs: https://www.getambassador.io/docs/edge-stack/latest/topics/running/running/#log-levels-and-debugging
    ### Goes to the Ambassdor Labs docs homepage
    a8r.io/documentation: https://www.getambassador.io/docs/latest/
    ### Emojivoto repository
    a8r.io/repository: https://github.com/aes-t2/dev-rel-01.git
    ### Ambassador Labs Support Portal Login Page
    a8r.io/support: https://support.datawire.io/hc/en-us
    ### Ambassador docs debugging page
    a8r.io/runbook: https://www.getambassador.io/docs/latest/topics/running/debugging/
    ### REQUIRES LOGIN: Datadog incident panel (Only shows demo screen)
    a8r.io/incidents: https://app.datadoghq.com/incidents/settings
    ### Jaeger Trace Boutique App.  Frontend to Shipping & Product Catalog
    a8r.io/uptime: https://monitoring.dr.mturner.k736.net/jaeger/trace/98ab4d0fa605c2f391340c299bfe2ef0
    ### Login before presenting. In most cases login is cached and not required each time. Username = admin, PW = admin
    a8r.io/performance: https://monitoring.dr.mturner.k736.net/grafana/d/R_NuxHVWk/ambassador-dashboard?refresh=1m&orgId=1
    a8r.io/dependencies: voting-svc.emojivoto, emoji-svc.emojivoto
```

## Telepresence demo

### Prerequisites

Telepresence 2.3.1 or later

Tested on MacOS 11.4 Big Sur.  Version number in parenthesis was the version tested (6/15/2021).

Requires membership to `a8r-dcp` organization (https://github.com/a8r-dcp/dev-rel-01/).  See `@mturner` for access.

- Docker (20.10.6)
- go (1.16.5)
- protobuf (3.17.3)
- protoc-gen-go (1.26.0)
- protoc-gen-go-grpc (1.1.0)
- yarn (1.22.10)

### Setup

Clone this repository and cd into folder:

```sh
git clone https://github.com/a8r-dcp/dev-rel-01
cd dev-rel-01
```

Export Kubeconfig:

```sh
export KUBECONFIG=$(pwd)/cluster-setup/dev-rel-01.yaml
```

Build dependencies and generate protobuf code:

```sh
make build
```

### Procedure

#### Show emojivoto-dev page

```sh
open emojivoto-dev.dr.mturner.k736.net
```

#### Show emojivoto-dev service catalog

```sh
open https://app.getambassador.io
```

Recommended: You can set some favorites (click the 3 dots on the right side of a service card) so that the dashboard isn't as cluttered.  These are managed on a per-user basis so they cannot be pre-configured by another person.

Select the `web-svc` card under the Development column.  This will be the deployment we will be intercepting with Telepresence.

#### Run intercept and start code locally

Start intercept:

```sh
telepresence login && \
telepresence intercept web-svc -n emojivoto-dev --port 8080:80 -u=true
```

*Note* Step 4 is required.  The Emojivoto app is virtually hosted and requires the specific hostname `emojivoto-dev.dr.mturner.k736.net` in order to be accessed.

```txt
  1/4: What's your ingress' layer 3 (IP) address?
    You may use an IP address or a DNS name (this is usually a "service.namespace" DNS name).
      [default: ambassador.ambassador]: ambassador.ambassador
  2/4: What's your ingress' layer 4 address (TCP port number)?
      [default: 443]: 443
  3/4: Does that TCP port on your ingress use TLS (as opposed to cleartext?
      [default: y]: y
  4/4: If required by your ingress, specify a different layer 5 hostname
    (TLS-SNI, HTTP "Host" header) to access this service.
      [default: ambassador.ambassador]: emojivoto-dev.dr.mturner.k736.net
```

Check out new test code:     #THIS NEEDS TO BE CHECKED WITH CASEY

```sh
git checkout cakuros/web-update
```

Open new terminal window, and start running the webpack-dev-server:  #THIS NEEDS TO BE CHECKED WITH CASEY

```sh
cd emojivoto-web
yarn webpack-dev-server --port 8083
```

Open new terminal window and start running the web server:  #THIS NEEDS TO BE CHECKED WITH CASEY

```sh
cd emojivoto-web
make run-tel
```

Open the new intercepted service (e.g. https://laughing-benz-205.preview-beta.edgestack.me):

```sh
open https://laughing-benz-205.preview-beta.edgestack.me
```

This build version has been pushed to `docker.io/caseykurosawa/emojivoto-{service-name}:v13-rc0` as a new image for Rollouts in the next section.

## Argo Rollouts demo

### Prerequisites

(Required)

- Enable Developer Preview mode in DCP.

  Open the Developer Control Plane, go to the web-svc card under the Production column.  Select the Rollouts tab and a popup will ask you if you want to enable the feature.  Select Enable Rollouts to continue.  This only needs to be done once per user.

(Optional)

- Kubectl argo rollout plugin (v1.0.0)

### Setup

Open ArgoCD dashboard (admin:devrel), click on the ac-web-svc-emojivoto app:

```sh
open https://argocd.dr.mturner.k736.net
```

In a new terminal window, monitor Argo Rollout status:

```sh
kubectl argo rollouts get rollout -n emojivoto web -w
```

### Procedure

Show the production version of Emojivoto:

```sh
emojivoto.dr.mturner.k736.net
```

Go back to DCP startpage:

```sh
open https://app.getambassador.io
```

Select the web-svc card under the Production column.

#### Click on the Start Rollout button in the top bar

Set the Image Tag to `caseykurosawa/emojivoto-web v13-rc0`, set your rollout duration, and weight increment.  Please keep the number of pods set to 2 at maximum (this is to avoid over-allocating CPU requests to the node).

*Note*: make sure to only click the `Start Rollout` button once.  Multiple clicks will result in multiple PR's being generated.

The button will automatically take you to the Rollouts tab and show a `Pull Request` button that you can click.  Follow the link and merge the PR.

Go back to the Argo dashboard and click Refresh for Argo to pick up the new change and initiate the rollout.

Back in the Service Catalog, you will be able to see some stats about the Rollout, including Canary percentage and the amount of time remaining in the rollout.

#### Check Grafana

Some grafana dashboards (admin:prom-operator):
Ambassador Dashboard: https://monitoring.dr.mturner.k736.net/grafana/d/R_NuxHVWk/ambassador-dashboard?refresh=1m&orgId=1
ArgoCD and ArgoRollouts Dashboards are TBD.  

#### Check new rollout

Once the rollout is complete, go back to the emojivoto app to see the new changes:

```sh
open https://emojivoto.dr.mturner.k736.net
```

## Revert back to prep for next demo

Re-run the same procedure creating a new rollout, but target `caseykurosawa/emojivoto-web v12` as the image version to restore the original emojivoto-web.

For Telepresence, run `telepresence quit` to disconnect all intercepts and disconnect from the cluster.

The running `make run-tel` and `yarn webpack-dev-server --port 8083` can be interrupted (Ctl + C)

Check out the main branch to get your local code back to the production version.

```sh
git checkout main
```
