# Axway Marketplace Subscription and Access Request Approval Workflow using API Builder and MS Teams

This API Builder project implements the steps described [**here**](https://docs.axway.com/bundle/amplify-central/page/docs/integrate_with_central/webhook/marketplace_subscription_webhook/index.html) to approve Axway Marketplace Product subscription approval requests.

This API Builder project also implements an approval flow for Axway Marketplace asset access requests.

A user requests access to a asset (resources) in order to retrieve credentials (e.g. API Key) to call the API.

It is intended to be a real world working example of a subscription workflow that relies on user intervention to approve/reject a request.

The end user flow for Product Subscription Requests looks like this:

![](https://i.imgur.com/RaRuLHh.png)

![](https://i.imgur.com/gaM3jcN.png)

![](https://i.imgur.com/zupyrCj.png)

![](https://i.imgur.com/BTP6bZ5.png)


The end user flow for Asset Requests is shown below:

![](https://i.imgur.com/ErQA2ud.png)

![](https://i.imgur.com/9ylWNXR.png)

![](https://i.imgur.com/4TRQVo0.png)

![](https://i.imgur.com/r3XbeWo.png)

![](https://i.imgur.com/KML7odK.png)


As should be evident in the screen shots above, one needs a Marketplace application and an approved product subscription in order to request access to a product's assets (resources).

The flow leverages MS Teams to present an approval form to the approver for a Product Subscription:

![](https://i.imgur.com/4ihUmHy.png)

and for an Access Request:

![](https://i.imgur.com/bUH5VTI.png)

The API Builder project exposes two API's:

* `POST /api/amplifycentralwebhookhandler` which takes an Amplify subscription webhook as the body. This is the webhook that Amplify calls when a marketplace product subscription request is made.
* `POST /api/approver` which is called automatically when the approver clicks the approve or reject button in the MS Teams approval form.

You need to set the following environment variables in your API Builder project (e.g. /conf/.env):

```
CLIENT_ID=<An Axway service account client ID>
CLIENT_SECRET=<An Axway service account client Secret>
API_KEY=<An API Key that you set>
API_CENTRAL_URL=https://apicentral.axway.com
MS_TEAMS_WEBHOOK_URL=<MS Teams channel incoing webhook URL>
BASEURL=<The base URL of your deployed API Builder project>
```

I configured my Marketplace Subscription Webhook as follows:

```
axway central create -f marketplaceapprovalworflowintegration.yaml
```

`marketplaceapprovalworflowintegration.yaml`

```
group: management
apiVersion: v1alpha1
kind: Integration
title: Integrations for Marketplace Subscription Approvals
name: integrations-for-subscription-approvals
spec:
  description: This is a group of resources to be used for marketplace subscription approval workflows
---
group: management
apiVersion: v1alpha1
kind: Webhook
title: Webhook Listener for Marketplace Subscription Approvals
name: wh-integrations-for-subscription-approvals
metadata:
  scope:
    kind: Integration
    name: integrations-for-subscription-approvals
spec:
  enabled: true
  url: <API Builder Base Address>/api/amplifycentralwebhookhandler
  headers:
      apikey: <API Key Set as Env Var>
      Content-Type: application/json
---
group: management
apiVersion: v1alpha1
kind: ResourceHook
title: Resource Hook for Marketplace Subscription Approvals
name: rh-integrations-for-subscription-approvals
metadata:
  scope:
    kind: Integration
    name: integrations-for-subscription-approvals
spec:
  triggers:
    - group: catalog
      kind: Subscription
      name: "*"
      type:
      - updated
  webhooks:
    - wh-integrations-for-subscription-approvals
```

> Note: You need to edit the YAML file above and set the API Builder base address and APIKey in the webhook section

The API Builder flow for `/amplifycentralwebhookhandler` is shown below:

![](https://i.imgur.com/lvLfikL.png)

The API Builder flow for `/approver` is shown below:

![](https://i.imgur.com/7ep1ogD.png)

The flows can both certainly be improved by handling errors and other HTTP response status codes and responses better but it demonstrates an MVP for now.

## Additional Features

When an approval is handled, an additional MS Teams message is sent to let the API Provider know that the request was processed:

![](https://i.imgur.com/qqnQqDV.png)

![](https://i.imgur.com/n2icLwM.png)

If a request is already processed, and an approver clicks the Approve/Reject button then the flow will not modify the processed approval and will notify via MS Teams:

![](https://i.imgur.com/oYDMWEk.png)

![](https://i.imgur.com/ieDLGSM.png)
