# Terraform Installation Guide

## Prerequisites

Before installing Terraform, ensure you have the following prerequisites:

- A supported operating system (Windows, macOS, Linux)
- An internet connection
- A user account with administrative privileges

## Installation Steps

### Windows

1. Download the Terraform binary from the [official Terraform website](https://www.terraform.io/downloads.html).
2. Extract the downloaded ZIP file to a directory of your choice.
3. Add the directory to your system's PATH environment variable:
    - Open the Start Search, type in "env", and select "Edit the system environment variables".
    - In the System Properties window, click on the "Environment Variables" button.
    - In the Environment Variables window, find the "Path" variable in the "System variables" section and click "Edit".
    - Click "New" and add the path to the directory where you extracted Terraform.
    - Click "OK" to close all windows.
4. Verify the installation by opening a new Command Prompt and running:
    ```sh
    terraform -v
    ```

### macOS

1. Install Terraform using Homebrew:
    ```sh
    brew tap hashicorp/tap
    brew install hashicorp/tap/terraform
    ```
2. Verify the installation by running:
    ```sh
    terraform -v
    ```

### Linux

1. Download the Terraform binary from the [official Terraform website](https://www.terraform.io/downloads.html).
2. Extract the downloaded ZIP file:
    ```sh
    unzip terraform_<VERSION>_linux_amd64.zip
    ```
3. Move the binary to a directory included in your system's PATH:
    ```sh
    sudo mv terraform /usr/local/bin/
    ```
4. Verify the installation by running:
    ```sh
    terraform -v
    ```

## Conclusion

You have successfully installed Terraform on your system. You can now start using Terraform to manage your infrastructure as code.

For more information and advanced usage, refer to the [official Terraform documentation](https://www.terraform.io/docs/index.html).