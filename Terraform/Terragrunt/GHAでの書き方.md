# GHAでのTerragrunt
>[GitHub ActionsでTerragruntとtfcmtを組み合わせてplan結果を見やすくする方法](https://qiita.com/tak0203753/items/5d12ea942ca12177238f)

```yaml
  deploy_infrastructure:
    needs: lambda_ci_cd
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
    - name: Checkout code
      uses: actions/checkout@v2
    # AWS OIDC認証
    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        role-to-assume: ${{ secrets.IAM_ROLE_ARN }}
        aws-region: ${{ env.AWS_REGION }}
    # Set up Terraform
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v1
    # Set up Terragrunt
    - name: Setup Terragrunt
      run: |
        sudo wget -q -O /bin/terragrunt "https://github.com/gruntwork-io/terragrunt/releases/download/v${TG_VERSION}/terragrunt_linux_amd64"
        sudo chmod +x /bin/terragrunt
        terragrunt --version

    - name: Terragrunt init
        working-directory: ${{ env.WORK_DIR }}
        run: terragrunt run-all init

    - name: Check terraform fmt
        working-directory: ${{ env.WORK_DIR }}
        run: terraform fmt -check -recursive

    - name: Terragrunt validate
        working-directory: ${{ env.WORK_DIR }}
        run: terragrunt run-all validate

    - name: Terragrunt plan
        working-directory: ${{ env.WORK_DIR }}
        env:
            GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
            terragrunt run-all plan --terragrunt-tfpath $GITHUB_WORKSPACE/.github/scripts/tfwrapper.sh
```
## stackoverflowでの回答
>[How to use Terragrunt in Github Actions](https://stackoverflow.com/questions/73182172/how-to-use-terragrunt-in-github-actions)

```yaml
name: 'Terragrunt CI'

on:
  push:
    branches:
    - main
  pull_request:

jobs:
  Terragrunt:
    name: 'Terragrunt'
    runs-on: ubuntu-latest

    # Use the Bash shell regardless whether the GitHub Actions runner is ubuntu-latest, macos-latest, or windows-latest
    defaults:
      run:
        shell: bash

    steps:
    # Checkout the repository to the GitHub Actions runner
    - name: Checkout
      uses: actions/checkout@v2

    # Install the latest version of Terragrunt CLI and configure the Terragrunt CLI configuration file with a Terragrunt Cloud user API token
    - name: Setup Terraform v1.2.6
      uses: hashicorp/setup-Terraform@v1
      with:
        terraform_version: 1.2.6
        terraform_wrapper: true
    - name: Setup Terraform version
      run: terraform --version
    - name: Setup Terraform wrapper path
      run: which terraform

    - name: Setup Terragrunt v0.38.4
      run: |
        sudo wget -q -O /bin/terragrunt "https://github.com/gruntwork-io/terragrunt/releases/download/v0.38.4/terragrunt_linux_amd64"
        sudo chmod +x /bin/terragrunt
        terragrunt -v

    # Initialize a new or existing Terragrunt working directory by creating initial files, loading any remote state, downloading modules, etc.
    - name: Terragrunt Init
      run: terragrunt init --terragrunt-non-interactive
      env:
        GOOGLE_CREDENTIALS: ${{ secrets.GOOGLE_CREDENTIALS }}

    # Generates an execution plan for Terragrunt
    - name: Terragrunt Plan
      run: terragrunt run-all plan --terragrunt-non-interactive
      env:
        GOOGLE_CREDENTIALS: ${{ secrets.GOOGLE_CREDENTIALS }}

      # On push to main, build or change infrastructure according to Terragrunt configuration files
      # Note: It is recommended to set up a required "strict" status check in your repository for "Terragrunt Cloud". See the documentation on "strict" required status checks for more information: https://help.github.com/en/github/administering-a-repository/types-of-required-status-checks
    - name: Terragrunt Apply
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
      run: terragrunt run-all apply --terragrunt-non-interactive
      env:
        GOOGLE_CREDENTIALS: ${{ secrets.GOOGLE_CREDENTIALS }}
```
## Setting Up CI/CD For Terragrunt With GitHub Actions
>[記事](https://blog.devops.dev/setting-up-ci-cd-for-terragrunt-with-github-actions-65684b42f2c4)
>[GH](https://github.com/vinycoolguy2015/terragrunt-ci-cd/tree/main)
