# GitOps Project Screenshot Guide

This guide provides a step-by-step walkthrough on **what**, **how**, and **when** to capture screenshots for your "GitOps on AWS" project article. These images will visually tell the story of your production-grade pipeline.

## 📋 Preparation
Before starting, ensure your environment is active and healthy:
1.  **EKS Cluster**: Active.
2.  **ArgoCD**: Port-forwarded or accessible via LoadBalancer.
3.  **Pipeline**: At least one successful execution completed.
4.  **App**: Running and accessible.

---

## 📸 Phase 1: The Foundation (Infrastructure)

**Goal**: Show that the underlying AWS infrastructure is provisioned and ready.

### 1. AWS EKS Cluster Status
*   **What**: The EKS Console showing your cluster details.
*   **How**: Go to AWS Console > EKS > Clusters. Click on `gitops-eks-cluster`.
*   **Focus**: The "Active" status and the Kubernetes version (v1.34).
*   **Caption**: "Provisioned EKS Cluster v1.34 running on Amazon Linux 2023."

### 2. ECR Repository
*   **What**: The Elastic Container Registry showing your Docker images.
*   **How**: Go to AWS Console > ECR > Repositories > `gitops-webapp`.
*   **Focus**: The list of images with "Image tags" corresponding to your Git commit hashes.
*   **Caption**: "ECR Repository hosting versioned application images."

### 3. CodeCommit Repository Structure
*   **What**: The file structure of your Git repository.
*   **How**: Go to AWS Console > CodeCommit > Repositories > `gitops-on-aws`.
*   **Focus**: The root directory showing `app/`, `gitops/`, `infrastructure/`, and `pipeline/` folders.
*   **Caption**: "Monorepo structure hosting Infrastructure, Application, and GitOps manifests."

---

## 📸 Phase 2: The CI Pipeline (Continuous Integration)

**Goal**: Demonstrate the automated build and "push" process.

### 4. CodePipeline Success View
*   **What**: The visual flow of your pipeline stages.
*   **How**: Go to AWS Console > CodePipeline > Pipelines > `gitops-demo-terraform-pipeline`.
*   **Focus**: Ensure both "Source" and "Build" stages are green (Succeeded).
*   **Caption**: "Automated CI Pipeline triggering on Git pushes."

### 5. CodeBuild "GitOps Magic" Log
*   **What**: The specific log lines where the pipeline updates the Git repository. This is the "secret sauce" of this project.
*   **How**: Click "View details" on the Build stage > Click the latest Build Run > "Build logs".
*   **When**: Scroll down to the `POST_BUILD` phase.
*   **Focus**: Highlight the lines:
    ```text
    Updating Helm chart values...
    ...
    git commit -m "Update image tag to..."
    git push origin main
    ```
*   **Caption**: "The CI pipeline automatically updating the Helm chart version in Git."

---

## 📸 Phase 3: The CD Controller (ArgoCD)

**Goal**: Show Kubernetes state management and synchronization.

### 6. ArgoCD Applications Dashboard
*   **What**: The main card view of ArgoCD.
*   **How**: Log in to your ArgoCD UI.
*   **Focus**: The `gitops-webapp` card showing a green heart (Healthy) and green checkmark (Synced).
*   **Caption**: "ArgoCD Dashboard monitoring the application state."

### 7. Application Tree View (The "Money Shot")
*   **What**: The visual hierarchy of Kubernetes resources.
*   **How**: Click on the `gitops-webapp` card in ArgoCD.
*   **Focus**: The tree showing: App -> Service -> Deployment -> ReplicaSet -> Pod.
*   **Caption**: "Visualizing the Kubernetes resources managed by ArgoCD."

---

## 📸 Phase 4: Verification & The Loop

**Goal**: Prove it works and show the full cycle.

### 8. The Running Application
*   **What**: The actual output of your deployed app.
*   **How**: Open the LoadBalancer URL in a browser OR run `curl` in your terminal.
*   **Focus**: The text: `Hello from GitOps on AWS! Running on pod: ...`
*   **Caption**: "The deployed application accessible via the AWS Load Balancer."

### 9. Automated Git History
*   **What**: The commit history showing the automated updates.
*   **How**: Go to AWS Console > CodeCommit > Repositories > `gitops-on-aws` > Commits.
*   **Focus**: The list of commits where the author is "CodeBuild" and the message is "Update image tag to...".
*   **Caption**: "Automated Git commits closing the GitOps loop."

---

## 💡 Pro Tips for Your Article
*   **Masking**: Blur out your AWS Account ID in the top right corner of AWS Console screenshots for privacy.
*   **Highlighting**: Use a red box or arrow to point to key elements (like the "Synced" status or the specific log line).
*   **Context**: Always include a bit of the surrounding interface so the reader knows where they are (e.g., keep the AWS header visible).
