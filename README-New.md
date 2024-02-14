# CI/CD using GitOps with Azure Arc-enabled Kubenetes

Please see the tutorial at <https://docs.microsoft.com/azure/azure-arc/kubernetes/tutorial-gitops-cicd>

https://learn.microsoft.com/en-us/azure/azure-arc/kubernetes/tutorial-gitops-flux2-ci-cd#implement-cicd-with-github

- arc-cicd-demo-src application repository
 - URL: https://github.com/Azure/arc-cicd-demo-src
 - Contains the example Azure Vote App that you will deploy using GitOps.

- arc-cicd-demo-gitops GitOps repository
 - URL: https://github.com/Azure/arc-cicd-demo-gitops
 - Works as a base for your cluster resources that house the Azure Vote App.

## variables
azureContainerRegistry="mtjacr001"
clusterName="aks01"
clusterRGName="aks-rg-01"
clusterConfigName="app-config-01"
gitopsNamespace="app-01"
gitopsRepo="https://github.com/manojtjacobs/gitops-code.git"
httpsUser="manojtjacobs"
httpsKey="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"
branch="master"
kustomizationName01="app-01-kustomize"

appRepo="application-code"
gitRepo="gitops-code"
organization="manojtjacobs"

# Create GitOps connection

az k8s-configuration flux create --name $clusterConfigName --cluster-name $clusterName --namespace $gitopsNamespace --resource-group $clusterRGName -u $gitopsRepo --https-user $httpsUser --https-key $httpsKey --scope cluster --cluster-type managedClusters --branch $branch --kustomization name=$kustomizationName01 prune=true path=vws-app --interval 2m

# Create GitOps connection/configuration
az k8s-configuration flux update --name $clusterConfigName --cluster-name $clusterName --resource-group $clusterRGName -u $gitopsRepo --https-user $httpsUser --https-key $httpsKey --cluster-type managedClusters --branch $branch --kustomization name=$kustomizationName01 prune=true path=testflux2/vws-app

# Show GitOps Configuration
az k8s-configuration flux show -g $clusterRGName -c $clusterName -n $clusterConfigName -t managedClusters

az aks update -n $clusterName -g $clusterRGName --attach-acr $azureContainerRegistry

# Install GitOps Connector
helm repo add gitops-connector https://azure.github.io/gitops-connector/
helm upgrade -i gitops-connector gitops-connector/gitops-connector \
      --namespace flux-system \
      --set gitRepositoryType=GITHUB \
      --set ciCdOrchestratorType=GITHUB \
      --set gitOpsOperatorType=FLUX \
      --set gitHubGitOpsRepoName=$appRepo \
      --set gitHubGitOpsManifestsRepoName=$gitRepo \
      --set gitHubOrgUrl=https://api.github.com/repos/$organization \
      --set gitOpsAppURL=https://github.com/$organization/$gitRepo/commit \
      --set orchestratorPAT=$httpsKey

# Run manifest in notification.md

## Create GitHub secrets

# Create GitHub repository secrets

AZURE_CREDENTIALS : {"clientId":"123456","clientSecret":"123546","subscriptionId":"123","tenantId":"123"}
AZ_ACR_NAME	: acrName
MANIFESTS_BRANCH : master
MANIFESTS_FOLDER : arc-cicd-cluster
MANIFESTS_REPO : https://github.com/$org/$repo
VOTE_APP_TITLE : Voting Application
AKS_RESOURCE_GROUP : aks-rg-01
AKS_NAME :	aks01
PAT : github_pat_12131
# Create GitHub environment secrets

#  az-vote-app-dev environment
ENVIRONMENT_NAME : Dev
TARGET_NAMESPACE : dev

# az-vote-app-stage environment
ENVIRONMENT_NAME : Stage
TARGET_NAMESPACE : stage