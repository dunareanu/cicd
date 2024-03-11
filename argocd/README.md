
# ArgoCD is a GitOps tool for deploying and managing applications on Kubernetes. Its components include:

* ArgoCD Server: The server component that provides a web UI and API for managing and monitoring application deployments.

* ArgoCD CLI: The command-line interface for interacting with the ArgoCD server.

* Application definition: YAML manifests that define an application's desired state.

* Git Repository: A source control repository containing the application definition and any other application assets.

* Sync Policy: Rules that determine how and when the desired state of an application should be synchronized with the actual state.

* Sync Loop: The process of comparing the desired and actual state of an application, and making any necessary changes to bring the actual state in line with the desired state.

* Role-based Access Control: A system that controls who can access and modify the ArgoCD server and applications.

* Audit Logs: A record of all changes and updates made to the ArgoCD server and applications.

# Procedures for adding APPS or REPOSITORIES:

* Go on GITLAB -> projects: https://gitlab.private.uae/gitlab/sysops ->  choose environment -> add application or repositories -> SYNC in "argo-itself-test" application

* A new repository or application will be configured


