# CI/CD using GitOps with Azure Arc-enabled Kubenetes

This repo contains the sample GitOps manifests for the Azure Arc GitOps CI/CD tutorial.

The GitOps repo can be found at <https://github.com/Azure/arc-cicd-demo-src>

Please see the tutorial at <https://docs.microsoft.com/azure/azure-arc/kubernetes/tutorial-gitops-cicd>

https://learn.microsoft.com/en-us/azure/azure-arc/kubernetes/tutorial-gitops-flux2-ci-cd#implement-cicd-with-github

azureContainerRegistry="mtjacr001"
clusterName="aks01"
clusterRGName="aks-rg-01"
clusterConfigName="cluster-config"
gitopsNamespace="cluster-config"
gitopsRepo="https://github.com/manojtjacobs/gitops-code.git"
httpsUser="manojtjacobs"
httpsKey="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"
branch="master"
kustomizationName01="cluster-config"

appRepo="application-code"
gitRepo="gitops-code"
organization="manojtjacobs"

az k8s-configuration flux create --name $clusterConfigName --cluster-name $clusterName --namespace $gitopsNamespace --resource-group $clusterRGName -u $gitopsRepo --https-user $httpsUser --https-key $httpsKey --scope cluster --cluster-type managedClusters --branch $branch --kustomization name=$kustomizationName01 prune=true path=arc-cicd-cluster/manifests

az k8s-configuration flux show -g $clusterRGName -c $clusterName -n $clusterConfigName -t managedClusters

az aks update -n $clusterName -g $clusterRGName --attach-acr $azureContainerRegistry

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

AZURE_CREDENTIALS : {"clientId":"ef9336c2-a64a-41a4-abc3-6ca760f94e87","clientSecret":"dsx8Q~Z7wFVIyO.TRcq9Mdol_bCzctYj.4TFVc31","subscriptionId":"82fa30a5-df4f-4f6d-9180-499dc53b772a","tenantId":"a196b33f-bb27-4684-bee1-b2ff40c77471"}
AZ_ACR_NAME	: mtjacr001
MANIFESTS_BRANCH : master
MANIFESTS_FOLDER : arc-cicd-cluster
MANIFESTS_REPO : https://github.com/manojtjacobs/gitops-code
VOTE_APP_TITLE : Voting Application
AKS_RESOURCE_GROUP : aks-rg-01
AKS_NAME :	aks01
PAT : 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

# Create GitHub environment secrets

#  az-vote-app-dev environment
ENVIRONMENT_NAME : Dev
TARGET_NAMESPACE : dev

# az-vote-app-stage environment
ENVIRONMENT_NAME : Stage
TARGET_NAMESPACE : stage