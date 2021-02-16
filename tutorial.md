# Using skaffold with a multi-repository application

## Introduction

### What is this project?
Forked from the original **[Bank of Anthos](https://github.com/GoogleCloudPlatform/bank-of-anthos)** project, it is a sample HTTP-based web app that simulates a bank's payment processing network, allowing users to create artificial bank accounts and complete transactions.

It consists of multiple microservices with each service code living in its own Github repository:
| Service(click to see repo)                                          | Language      | Description                                                                                                                                  |
| ------------------------------------------------ | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| [frontend](https://github.com/gsquared94/bank-of-anthos-frontend)                       | Python        | Exposes an HTTP server to serve the website. Contains login page, signup page, and home page.                                                |
| [ledger-writer](https://github.com/gsquared94/bank-of-anthos-ledgerwriter)              | Java          | Accepts and validates incoming transactions before writing them to the ledger.                                                               |
| [balance-reader](https://github.com/gsquared94/bank-of-anthos-balancereader)            | Java          | Provides efficient readable cache of user balances, as read from `ledger-db`.                                                                |
| [transaction-history](https://github.com/gsquared94/bank-of-anthos-transactionhistory)  | Java          | Provides efficient readable cache of past transactions, as read from `ledger-db`.                                                            |
| [ledger-db](https://github.com/gsquared94/bank-of-anthos-ledger-db)                     | PostgreSQL | Ledger of all transactions. Option to pre-populate with transactions for demo users.                                                         |
| [user-service](https://github.com/gsquared94/bank-of-anthos-userservice)                | Python        | Manages user accounts and authentication. Signs JWTs used for authentication by other services.                                              |
| [contacts](https://github.com/gsquared94/bank-of-anthos-contacts)                       | Python        | Stores list of other accounts associated with a user. Used for drop down in "Send Payment" and "Deposit" forms. |
| [accounts-db](https://github.com/gsquared94/bank-of-anthos-accounts)                 | PostgreSQL | Database for user accounts and associated data. Option to pre-populate with demo users.                                                      |
| [loadgenerator](https://github.com/gsquared94/bank-of-anthos-loadgenerator)             | Python/Locust | Continuously sends requests imitating users to the frontend. Periodically creates new accounts and simulates transactions between them.      |

### What you'll learn

- how to build and deploy a multi-repository application using `skaffold`
- how to iterate on individual services or groups of services.

___

**Time to complete**: <walkthrough-tutorial-duration duration=15></walkthrough-tutorial-duration>

Click the **Start** button to move to the next step.

## First steps

### Install Skaffold

Check the version of `skaffold` installed.

Run:
```bash
skaffold version
```

 If it is older than `v1.20.0`, then we manually install it. 

Run:
```bash
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/v1.20.0/skaffold-linux-amd64 
sudo install skaffold /usr/bin/skaffold
```

### Start a Minikube cluster

We'll use `minikube` as our local kubernetes cluster of choice.

Run:
```bash
minikube start
```


## Build and deploy all services

The root <walkthrough-editor-open-file filePath="skaffold.yaml">skaffold.yaml</walkthrough-editor-open-file> file lists all the services as `git` dependencies using the `requires` stanza. They are composed into three `modules` (compare with the <walkthrough-editor-open-file filePath="architecture.png">architecture diagram</walkthrough-editor-open-file>):
- <walkthrough-editor-select-line filePath="skaffold.yaml" startLine="3" startCharacterOffset="7" endLine="4" endCharacterOffset="0">frontend-svc</walkthrough-editor-select-line> adds the `frontend` service.
- <walkthrough-editor-select-line filePath="skaffold.yaml" startLine="17" startCharacterOffset="7" endLine="18" endCharacterOffset="0">loadgenerator-svc</walkthrough-editor-select-line> adds the `loadgen` service.
- <walkthrough-editor-select-line filePath="skaffold.yaml" startLine="28" startCharacterOffset="7" endLine="29" endCharacterOffset="0">backend-svc</walkthrough-editor-select-line> adds all the other services and databases dealing with `accounts` and `transactions`.

Launching `skaffold` with this file will clone all the specified projects into skaffold's cache directory (`~/.skaffold/repos` by default) and import them as skaffold artifacts in a single session. Each service repository defines its own `skaffold.yaml` configuration (eg. [accounts-db/skaffold.yaml](https://github.com/gsquared94/bank-of-anthos-accounts/blob/main/skaffold.yaml)) that instructs `skaffold` how to build and deploy it. 

Run:
```bash
skaffold dev --port-forward --tail=false
```

Once all images are built (might take a while the first time) and deployed (check for streaming log message `"Deployments stabilized in x.xx seconds"`) click on the <walkthrough-web-preview-icon></walkthrough-web-preview-icon> icon and select `Change Port` and change the preview port to `4503`. This should redirect to the frontend service and show the running application.

<walkthrough-footnote>
    `--port-forward` flag forwards local ports to all service ports.
    `--tail=false` supresses streaming container logs
</walkthrough-footnote>

## Develop only selected services

We can scope each `skaffold` session to only selected modules by specifying the target config names with the `-m` or `--module` flag.

To only run the `frontend` and `backend` modules without the artificial load generator service, run:

Run:
```bash
skaffold dev -m frontend-svc --port-forward --tail=false
```

Images should already be available in the `skaffold` cache. Once deployed click on the <walkthrough-web-preview-icon></walkthrough-web-preview-icon> icon and click `Preview on port 4503`. This should redirect to the frontend service and show the running application as before.

<walkthrough-footnote>
    To run only the `backend` services, use the `-m backend-svc` flag. To iterate on an individual backend service, use the config name defined in that service's `skaffold.yaml` directly. 
</walkthrough-footnote>

## Congratulations

<walkthrough-conclusion-trophy></walkthrough-conclusion-trophy>

All done!

You now know how to use _remote config dependencies_ in Skaffold to develop multi-repository applications.
