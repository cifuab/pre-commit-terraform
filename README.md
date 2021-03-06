# Collection of git hooks for Terraform to be used with [pre-commit framework](http://pre-commit.com/)

[![Github tag](https://img.shields.io/github/tag/antonbabenko/pre-commit-terraform.svg)](https://github.com/antonbabenko/pre-commit-terraform/releases) ![](https://img.shields.io/maintenance/yes/2020.svg) [![Help Contribute to Open Source](https://www.codetriage.com/antonbabenko/pre-commit-terraform/badges/users.svg)](https://www.codetriage.com/antonbabenko/pre-commit-terraform)

## How to install

### 1. Install dependencies

* [`pre-commit`](https://pre-commit.com/#install)
* [`terraform-docs`](https://github.com/segmentio/terraform-docs) required for `terraform_docs` hooks. `GNU awk` is required if using `terraform-docs` older than 0.8.0 with Terraform 0.12.
* [`TFLint`](https://github.com/terraform-linters/tflint) required for `terraform_tflint` hook.
* [`TFSec`](https://github.com/liamg/tfsec) required for `terraform_tfsec` hook.
* [`coreutils`](https://formulae.brew.sh/formula/coreutils) required for `terraform_validate` hook on macOS (due to use of `realpath`).

##### MacOS

```bash
brew tap liamg/tfsec
brew install pre-commit gawk terraform-docs tflint tfsec coreutils
```

##### Ubuntu

```bash
sudo apt install python3-pip gawk &&\
pip3 install pre-commit
curl -L "$(curl -s https://api.github.com/repos/segmentio/terraform-docs/releases/latest | grep -o -E "https://.+?-linux-amd64")" > terraform-docs && chmod +x terraform-docs && sudo mv terraform-docs /usr/bin/
curl -L "$(curl -s https://api.github.com/repos/terraform-linters/tflint/releases/latest | grep -o -E "https://.+?_linux_amd64.zip")" > tflint.zip && unzip tflint.zip && rm tflint.zip && sudo mv tflint /usr/bin/
env GO111MODULE=on go get -u github.com/liamg/tfsec/cmd/tfsec
```

### 2. Install the pre-commit hook globally

```bash
DIR=~/.git-template
git config --global init.templateDir ${DIR}
pre-commit init-templatedir -t pre-commit ${DIR}
```

### 3. Add configs and hooks

Step into the repository you want to have the pre-commit hooks installed and run:

```bash
git init
cat <<EOF > .pre-commit-config.yaml
repos:
- repo: git://github.com/antonbabenko/pre-commit-terraform
  rev: <VERSION> # Get the latest from: https://github.com/antonbabenko/pre-commit-terraform/releases
  hooks:
    - id: terraform_fmt
    - id: terraform_docs
EOF
```

### 4. Run

After pre-commit hook has been installed you can run it manually on all files in the repository

```bash
pre-commit run -a
```

## Available Hooks

There are several [pre-commit](https://pre-commit.com/) hooks to keep Terraform configurations (both `*.tf` and `*.tfvars`) and Terragrunt configurations (`*.hcl`) in a good shape:

| Hook name                                        | Description                                                                                                                |
| ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| `terraform_fmt`                                  | Rewrites all Terraform configuration files to a canonical format.                                                          |
| `terraform_validate`                             | Validates all Terraform configuration files.                                                                               |
| `terraform_docs`                                 | Inserts input and output documentation into `README.md`. Recommended.                                                      |
| `terraform_docs_without_aggregate_type_defaults` | Inserts input and output documentation into `README.md` without aggregate type defaults.                                   |
| `terraform_docs_replace`                         | Runs `terraform-docs` and pipes the output directly to README.md                                                           |
| `terraform_tflint`                               | Validates all Terraform configuration files with [TFLint](https://github.com/terraform-linters/tflint).                              |
| `terragrunt_fmt`                                 | Rewrites all [Terragrunt](https://github.com/gruntwork-io/terragrunt) configuration files (`*.hcl`) to a canonical format. |
| `terraform_tfsec`                                | [TFSec](https://github.com/liamg/tfsec) static analysis of terraform templates to spot potential security issues.     |

Check the [source file](https://github.com/antonbabenko/pre-commit-terraform/blob/master/.pre-commit-hooks.yaml) to know arguments used for each hook.

## Notes about terraform_docs hooks

1. `terraform_docs` and `terraform_docs_without_aggregate_type_defaults` will insert/update documentation generated by [terraform-docs](https://github.com/segmentio/terraform-docs) framed by markers:
```txt
<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
```
if they are present in `README.md`.

1. `terraform_docs_replace` replaces the entire README.md rather than doing string replacement between markers. Put your additional documentation at the top of your `main.tf` for it to be pulled in. The optional `--dest` argument lets you change the name of the file that gets created/modified.

    1. Example:
    ```yaml
    hooks:
      - id: terraform_docs_replace
        args: ['--with-aggregate-type-defaults', '--sort-inputs-by-required', '--dest=TEST.md']
    ```

1. It is possible to pass additional arguments to shell scripts when using `terraform_docs` and `terraform_docs_without_aggregate_type_defaults`. Send pull-request with the new hook if there is something missing.

## Notes about terraform_tflint hooks

1. `terraform_tflint` supports custom arguments so you can enable module inspection, deep check mode etc.

    1. Example:
    ```yaml
    hooks:
      - id: terraform_tflint
        args: ['args=--deep']
    ```

    In order to pass multiple args, try the following:
    ```yaml
     - id: terraform_tflint
       args:
          - 'args=--deep'
          - 'args=--enable-rule=terraform_documented_variables'
    ```

## Notes about terraform_tfsec hooks

1. `terraform_tfsec` will recurse all directories/modules.
1. To ignore specific warnings, follow the convention from the
[documentation](https://github.com/liamg/tfsec#ignoring-warnings).
    1. Example:
    ```hcl
    resource "aws_security_group_rule" "my-rule" {
        type = "ingress"
        cidr_blocks = ["0.0.0.0/0"] #tfsec:ignore:AWS006
    }
    ```


## Notes for developers

1. Python hooks are supported now too. All you have to do is:
    1. add a line to the `console_scripts` array in `entry_points` in `setup.py`
    1. Put your python script in the `pre_commit_hooks` folder

Enjoy the clean and documented code!

## Authors

This repository is managed by [Anton Babenko](https://github.com/antonbabenko) with help from [these awesome contributors](https://github.com/antonbabenko/pre-commit-terraform/graphs/contributors).

## License

MIT licensed. See LICENSE for full details.
