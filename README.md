# Konfigurasi Marketplace Batam

Gunakan [panduan setup marketplace](SETUP-MARKETPLACE.md) dan [panduan cron-job.org](SETUP-CRONJOB.md) untuk fork ini. Workflow memakai A1.Flex 1 OCPU, 4 GB RAM, boot volume 50 GB, dan satu percobaan per pemicu dari cron-job.org setiap menit. Instruksi umum dari repo asal di bawah dapat berbeda dari konfigurasi ini.

---

# OCI Always Free Arm (A1.Flex) VM Creator

[![Try to Create OCI VM](https://github.com/heykapil/oci-free-arm-instance/actions/workflows/create-vm.yml/badge.svg)](https://github.com/heykapil/oci-free-arm-instance/actions/workflows/create-vm.yml)

This repository contains a GitHub Actions workflow that automatically tries to provision an "Always Free" `VM.Standard.A1.Flex` (Arm) compute instance in your Oracle Cloud Infrastructure (OCI) account.

This is necessary because the "Always Free" Arm instances are a popular resource and are often unavailable due to high demand, resulting in an `"Out of host capacity."` error. You can try to upgrade to `pay as you go` plan which has a very good chance of getting available instance. Be sure to remain in free limits and check your costs frequently.

## Features

* **Fully Automated:** cron-job.org triggers GitHub Actions. You don't need to run anything on your local machine.
* **Persistent:** For this fork, cron-job.org triggers the workflow each minute until it successfully provisions your VM.
* **Secure:** OCI credentials stay in GitHub Secrets. cron-job.org stores a GitHub token scoped to this repository with Actions write access. The repository itself contains no private information.
* **Fast:** Uses GitHub's caching to store the `oci-cli` installation, so subsequent runs are much faster.
* **Informative:** Sends Discord notifications when a VM is created or found, or when an unexpected error occurs. Capacity errors appear in GitHub Actions logs.

---

## How to Use

To use this, you need to **Fork** this repository and set up your OCI credentials as GitHub Secrets.

### Prerequisites

* An Oracle Cloud Infrastructure (OCI) "Always Free" account.
* A GitHub account.
* A Discord server/channel to receive notifications.

---

## Step 1: Fork This Repository

Click the **"Fork"** button at the top-right of this page. This will create a copy of this repository in your own GitHub account. All the following steps will be done on **your fork**.

## Step 2: Generate Your OCI API Credentials

This is the most detailed step. You need to gather **11 pieces of information** from your OCI account and your computer.

### A. Core IDs (Tenancy, User, Region)

1.  Log in to your OCI Console.
2.  **Tenancy OCID:** Click your **Profile icon** (top right) -> **Tenancy: [your\_tenancy\_name]**.
    * Copy the **OCID** value. This is your `OCI_CLI_TENANCY`.
    * *Note: For "Always Free" accounts, this is also your `COMPARTMENT_ID`.*
3.  **User OCID:** Click your **Profile icon** -> **User Settings**.
    * Copy the **OCID** value. This is your `OCI_CLI_USER`.
4.  **Region Identifier:** Look in the top-right corner of the console (e.g., "Singapore").
    * Hover over it or click it to find the identifier (e.g., `ap-singapore-1`). This is your `OCI_CLI_REGION`.

### B. OCI API Key (Private Key & Fingerprint)

1.  On the same **User Settings** page, click **"API Keys"** from the left-hand menu.
2.  Click the **"Add API Key"** button.
3.  Select **"Generate API Key Pair"**.
4.  Click **"Download Private Key"** and save the `oci_api_key.pem` file. **Do not lose this file.**
5.  Click the **"Add"** button.
6.  A "Configuration File Preview" will pop up. From this box, copy the `fingerprint` value. This is your `OCI_CLI_FINGERPRINT`.

### C. VM-Specific IDs (Subnet, Image, AD)

1.  **Subnet ID:**
    * Go to the OCI Console menu (☰) -> **Networking** -> **Virtual Cloud Networks**.
    * Click on your VCN (there is likely one default VCN).
    * Click on **"Subnets"** in the left menu.
    * Click on your public subnet (e.g., `Public Subnet ...`).
    * Copy the **OCID** of the subnet. This is your `SUBNET_ID`.
2.  **Availability Domain (AD) Name:**
    * Go to the OCI Console menu (☰) -> **Compute** -> **Instances**.
    * Click **"Create Instance"**.
    * In the **"Placement"** section, look at the **"Availability Domain"** dropdown. You likely only have one.
    * Copy its name *exactly* as it appears (e.g., `KClJ:AP-SINGAPORE-1-AD-1`). This is your `AD_NAME`.
3.  **Image ID:**
    * On the same "Create Instance" page, in the **"Image and shape"** section, click **"Change Image"**.
    * Select **"Canonical Ubuntu"** (or another OS of your choice).
    * Click the name of the image (e.g., "Canonical Ubuntu 22.04"). A details panel will slide out.
    * Copy the **OCID** of the image. This is your `IMAGE_ID`.
    * You can now cancel the "Create Instance" wizard.
    * If you still can not find the image, check this website and choose your image and copy ocid mentioned according to region. <https://docs.oracle.com/en-us/iaas/images/>
    * set this `IMAGE_ID` in your action env.
4.  **OCPUs and RAM**
    * The settings in this fork provision an instance with 1 OCPU and 4 GB memory.
    * Change `OCPUS` and `MEMORY_GB` in `.github/workflows/create-vm.yml` to adjust this.
6. **Boot Volume and Name**
    * This fork provisions a 50 GB boot volume named `marketplace-a1`.
    * Change `BOOT_VOLUME_GB` and `INSTANCE_NAME` in `.github/workflows/create-vm.yml` to adjust these values.
   
### D. Your SSH Public Key

This is the key you will use to log in to your new server.

1.  Open a terminal on your computer.
2.  Check if you have a key: `cat ~/.ssh/id_rsa.pub`
3.  **If it shows a key:** Copy the entire output (it starts with `ssh-rsa...`). This is your `SSH_PUBLIC_KEY`.
4.  **If it shows "No such file":** Run `ssh-keygen -t rsa -b 2048`. Press Enter three times to accept the defaults. Then, run `cat ~/.ssh/id_rsa.pub` again and copy the key.

---

## Step 3: Create a Discord Webhook

1.  Open your Discord server. Right-click on a channel name and click **"Edit Channel"**.
2.  Go to the **"Integrations"** tab.
3.  Click **"Webhooks"** -> **"New Webhook"**.
4.  Give it a name (e.g., "OCI Notifier") and click **"Copy Webhook URL"**.

---

## Step 4: Configure GitHub Secrets

Go to your forked repository on GitHub.

1.  Click the **"Settings"** tab.
2.  In the left menu, click **"Secrets and variables"** -> **"Actions"**.
3.  Click the **"New repository secret"** button for *each* of the 11 secrets listed below.

#### **VM Secrets**

* `OCI_COMPARTMENT_ID` (Value: Your Tenancy OCID from Step 2A)
* `IMAGE_ID` (Value: Your Image OCID from Step 2C)
* `OCI_SUBNET_ID` (Value: Your Subnet OCID from Step 2C)
* `AD_NAME` (Value: Your Availability Domain name from Step 2C)
* `SSH_PUBLIC_KEY` (Value: The `ssh-rsa...` key from Step 2D)

#### **Authentication Secrets**

* `OCI_CLI_REGION` (Value: Your region from Step 2A, e.g., `ap-singapore-1`)
* `OCI_CLI_USER` (Value: Your User OCID from Step 2A)
* `OCI_CLI_TENANCY` (Value: Your Tenancy OCID from Step 2A)
* `OCI_CLI_FINGERPRINT` (Value: The fingerprint from Step 2B)
* `OCI_CLI_KEY_CONTENT` (Value: Open the `oci_api_key.pem` file you downloaded in Step 2B with a text editor. Copy the *entire contents*, from `-----BEGIN PRIVATE KEY-----` to `-----END PRIVATE KEY-----`, and paste it here.)

#### **Notification Secret**

* `DISCORD_WEBHOOK_URL` (Value: The URL you copied from Discord in Step 3)

---

## Step 5: Run the Workflow

You're all set! Now you just need to start the process.

1.  Go to the **"Actions"** tab of your forked repository.
2.  In the left sidebar, click on **"OCI A1 Batam Capacity Catcher"**.
3.  You will see a message: "This workflow has a `workflow_dispatch` event." Click the **"Run workflow"** button on the right, and then **"Run workflow"** again.

This will start the first run. For this fork, follow [the cron-job.org setup](SETUP-CRONJOB.md) to trigger future runs each minute. Check the "Actions" tab for logs; Discord sends notifications when the VM is created, an instance already exists, or an unexpected error occurs.

---

## 🚨 CRITICAL: What to Do on Success

When Discord reports that the VM was created, verify the instance in the OCI Console.

The workflow attempts to disable the cron-job.org job and itself automatically. Verify both are disabled:

1.  Go to the **"Actions"** tab in your repository.
2.  Click on **"OCI A1 Batam Capacity Catcher"** in the sidebar.
3.  Click the **three-dot (...)** menu on the right.
4.  If still active, click **"Disable workflow"**.

Also check the job in cron-job.org and disable it manually if it is still active. A disabled GitHub workflow does not turn off an external cron job by itself.

Your VM will be provisioning in the OCI console. You can now log in using the SSH key you provided.

## License

This repository is available under the [MIT License](LICENSE).
