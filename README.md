# GitOps on AWS: ArgoCD + EKS + CodeCommit/CodePipeline

This project demonstrates a production-grade GitOps workflow on AWS using ArgoCD, Amazon EKS, and AWS CodeCommit/CodePipeline.

## Architecture

1.  **Infrastructure**: Provisioned via Terraform (VPC, EKS, ECR).
2.  **Source Code**: Hosted in AWS CodeCommit.
3.  **CI Pipeline**: AWS CodePipeline + CodeBuild builds the Docker image, pushes to ECR, and updates the Helm chart in the Git repo.
4.  **CD Controller**: ArgoCD running in EKS monitors the Git repo and syncs changes to the cluster.

## Prerequisites

-   AWS CLI configured with Administrator permissions.
-   Terraform installed (v1.0+).
-   kubectl installed.
-   Helm installed.
-   Git installed.

## Project Structure

-   `infrastructure/terraform`: Terraform code for AWS resources.
-   `app/`: Sample Python Flask application.
-   `gitops/`: Helm charts and ArgoCD application manifests.
-   `pipeline/`: Buildspec for CodeBuild.

## Deployment Steps

### 1. Provision Infrastructure

Navigate to the terraform directory and apply the configuration:

```bash
cd infrastructure/terraform
terraform init
terraform apply -auto-approve
```

This will create the VPC, EKS cluster, and ECR repository. Note the outputs (Cluster Name, ECR URL).

### 2. Configure kubectl

Update your kubeconfig to interact with the new cluster:

```bash
aws eks update-kubeconfig --region <REGION> --name <CLUSTER_NAME>
```

### 3. Install ArgoCD

Install ArgoCD into the cluster:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Access the ArgoCD UI:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Login with username 'admin' and password (get it via kubectl)
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

### 4. Setup CodeCommit and Push Code

The Terraform configuration has already created the CodeCommit repository `gitops-on-aws`. You just need to push the code.

(If you haven't already pushed the code as part of the setup script):

```bash
git remote add origin https://git-codecommit.us-west-2.amazonaws.com/v1/repos/gitops-on-aws
git push -u origin main
```

### 5. Connect ArgoCD to CodeCommit

Since CodeCommit repositories are private, you must provide credentials to ArgoCD.

1.  **Generate Git Credentials**:
    *   Go to the AWS IAM Console -> Users -> Your User -> Security Credentials.
    *   Scroll to "HTTPS Git credentials for AWS CodeCommit" and click "Generate credentials".
    *   Download or copy the Username and Password.

2.  **Add Repository to ArgoCD**:
    *   Access the ArgoCD UI (see step 3).
    *   Go to **Settings** -> **Repositories** -> **Connect Repo**.
    *   **Method**: HTTPS
    *   **Type**: Helm
    *   **Project**: default
    *   **Repository URL**: `https://git-codecommit.us-west-2.amazonaws.com/v1/repos/gitops-on-aws`
    *   **Username**: (Your generated username)
    *   **Password**: (Your generated password)
    *   Click **Connect**.
    *   *Note*: You also need to add the same repo as type "Git" so ArgoCD can monitor the Application manifest if you were storing it there, but here we are pointing to the Helm chart path. Add it as type **Git** as well with the same credentials.

### 6. Setup CodePipeline

The pipeline is already provisioned via Terraform! It triggers automatically on commits to the `main` branch.

You can view the pipeline status in the [AWS CodePipeline Console](https://us-west-2.console.aws.amazon.com/codesuite/codepipeline/pipelines/gitops-demo-terraform-pipeline/view).

### 7. Deploy Application via ArgoCD

Apply the ArgoCD Application manifest:

```bash
kubectl apply -f gitops/argocd-app.yaml
```

ArgoCD will now sync the Helm chart from the `gitops/charts/webapp` directory to the `default` namespace.

## GitOps Workflow

1.  Make a change to `app/app.py`.
2.  Commit and push to `main`.
3.  CodePipeline triggers -> Builds new image -> Pushes to ECR -> Updates `values.yaml` with new tag -> Commits back to repo.
4.  ArgoCD detects the change in `values.yaml` and syncs the new image to the EKS cluster.

## Cleanup

To destroy the infrastructure:

```bash
cd infrastructure/terraform
terraform destroy -auto-approve
```
