 # My Homelab

  This repository documents the setup for my personal homelab. The core of the lab
  is a lightweight k3s cluster, managed using GitOps principles.

  The primary goal is to create a reproducible and easy-to-manage Kubernetes
  environment for learning and experimentation. While my personal admin machine is
  declaratively configured with Nix, the server, following the same declarative philosophies uses a GitOps workflow

  #### *Core Components*

   * Host OS: DietPi (https://dietpi.com/) (on a Raspberry Pi 4)
   * Container Orchestration: k3s (https://k3s.io/), a lightweight and certified
     Kubernetes distribution.
   * GitOps Agent: Argo CD (https://argo-cd.readthedocs.io/en/stable/), to
     automatically sync the state of the cluster with this Git repository.
   * Admin Environment: My personal machine, configured with Nix & Home Manager
     (https://github.com/nix-community/home-manager) using the files in this
     repository.

  ### Setup Process

  This setup involves & assumes you have a cluster already running. 
  If you don't, do that first before continuing.
   
  1. Local Machine (Admin) Setup
   - Install Tools: kubectl and other necessary tools are installed via Home
      Manager. See the home.packages list in home-manager/common/default.nix.

   - Configure `kubectl`: To connect to the Pi's k3s cluster, we need its
      configuration file (kubeconfig).
       * First, retrieve the kubeconfig from the Pi: sudo cat
         /etc/rancher/k3s/k3s.yaml
       * Save this content on the local machine to a new file, e.g.,
         ~/.kube/k3s-pi.yaml.
       * Crucially, edit this new file and change the server address from
         https://127.0.0.1:6443 to the Pi's actual IP address.
       * Tell kubectl to merge this config file with any others by setting an
         environment variable. This can be added to your shell's startup file.
          `export KUBECONFIG=~/.kube/config:~/.kube/k3s-pi.yaml`

  2. GitOps Agent Installation

  Argo CD is installed into the cluster to manage all other applications. This is
  done from the local machine using kubectl.
  
  To install Argo, run the just command: 
  ```
  just install-argo
  ```
   
  ### GitOps Workflow
  With the server and GitOps agent in place on your cluster, all future applications 
  are deployed by the following the GitOps loop:

   1. Define: Create standard Kubernetes YAML manifests (Deployments, Services, etc.)
      for an application and place them in a designated directory within this
      repository.
   2. Commit & Push: Commit the new manifests to this repository and push the
      changes.
   3. Sync: Argo CD automatically detects the changes in the Git repository and
      applies them to the Kubernetes cluster, ensuring the cluster's state always
      matches the "desired state" defined in Git.
