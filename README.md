## **Purpose**
#### This repository contains basic terraform configurations for provisioning a new GCP project. ####

1: Create a git repo locally and clone it to GitHub using gh cli
```
    gh repo create <gcp-project-01> --public -c && cd <gcp-project-01>
```

2: Copy all files (*.tf, any *.tbd (to be deployed), except .terraform and .terraform.local.hcl) to the new folder. 

3: Open vscode.  
   Click on backend.tf and update terraform workspace name (yet to be created) and verify organization name.
   Backend is required for executing `terraform plan` locally for verifying code, syntax, and view plan output before committing and pushing code.

```
terraform {
  backend "remote" {
    hostname = "app.terraform.io"
    organization = "ramantandon"
    workspaces {
      name = "<gcp-project-01>"
    }
  }
}
```

4: Execute `terraform login` which stores token for TFC under `~/.terraform.d/credentials.tfrc.json`  
```
# terraform login
Terraform will request an API token for app.terraform.io using your browser.
If login is successful, Terraform will store the token in plain text in
the following file for use by subsequent commands:
    /root/.terraform.d/credentials.tfrc.json
Do you want to proceed?
  Only 'yes' will be accepted to confirm.
  Enter a value: yes
------------------------------------------------------------------------
Open the following URL to access the tokens page for app.terraform.io:
    https://app.terraform.io/app/settings/tokens?source=terraform-login
------------------------------------------------------------------------
Generate a token using your browser, and copy-paste it into this prompt.
Terraform will store the token in plain text in the following file
for use by subsequent commands:
    /root/.terraform.d/credentials.tfrc.json
Token for app.terraform.io:
  Enter a value:
Retrieved token for user rtandon
------------------------------------------------------------------------
Success! Terraform has obtained and saved an API token.
The new API token will be used for any future Terraform command that must make
authenticated requests to app.terraform.io.
```

5: Execute `terraform plan`. `terraform plan` will download required provider versions  
and create `terraform.tfstate` under `.terraform` directory. It will also create dependency lock file `.terraform.lock.hcl`  
I would suggest to include and commit these dot files, so we have uniformity in provider versions and dependency lock.

6: Update project name, billing account id in variables.tf
  Verify other terraform configs main.tf, network.tf, versions.tf, providers.tf

7: Git add, commit, and push code to remote repository.

8: Terraform Cloud Workflow Steps (creating workspace, Selecting VCS branch)
  - Login to [Terraform Cloud](https://app.terraform.io/)
  - Click New Workspace
  - Choose Version Control Workflow
  - Connect to version control provider (ex: GitHub App, BitBucket Cloud)
  - Filter and select your repository
  - Verify or update Workspace Name
  - Click Advanced Options, and type in VCS branch, as applicable.
  - Create Workspace
  - Click on Variables tab, and add an Environment Variable
  - Key would be `GOOGLE_CREDENTIALS` and value would be `application_default_credentials.json` 
  - Note: `GOOGLE_CREDENTIALS` value can be either `application_default_credentials.json` created by running 
    `gcloud    auth application-default login` or it can be private key of a Service Account, but if you are not having a GCP  Organisation and only creating non-org projects then you cannot create a project using terraform's `google_project` resource using a service account key. In that case, it needs to be user account.
  - Note: You simply **CANNOT** copy/paste keys to Terraform's env. variable as there are newline characters added. 
    To remove newline characters, we can use vscode. Copy key in vscode, select the key, ctrl + shift + p, Join Lines => enter. Copy the resultant key and paste it in TFC.

9: Execute first plan/apply manually at console. Subsequent code changes will automatically trigger terraform plan.
   You can go to Settings in Terraform console to auto apply, as required.

## Nice commands to know
- `terraform validate`  - validates the syntax and arguments of Terraform configuration files
- `terraform fmt`       - rewrites Terraform configuration files to a canonical format and style
- `terraform providers` - shows information about the provider requirements of the configutaion in the current working directory
- `terraform console`   - provides and interactive command-line console for evaluating and experimenting with expressions. Useful for testing interpolations.
