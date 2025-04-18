# gemini-proxy-panel

[中文介绍](./README_zh.md "Chinese Readme") <br>
[![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/dreamhartley/gemini-proxy-panel)

## Introduction

`gemini-proxy-panel` is a proxy service deployed on Cloudflare Workers. It forwards requests formatted for the OpenAI API to the Google Gemini Pro API, allowing applications developed for OpenAI to seamlessly switch to or leverage the capabilities of Gemini models.

## Features

*   **OpenAI to Gemini Proxy**: Seamlessly translates OpenAI Chat API requests into Gemini Pro API requests.
*   **Multi-API Key Rotation**: Supports configuring multiple Gemini API keys and automatically rotates through them to distribute request load and circumvent rate limits.
*   **Quota and Usage Management**: Monitor the usage of each Gemini API key through an intuitive management interface.
*   **Key Management**: Centrally manage multiple Gemini API keys and Worker API keys (used to access this proxy service) within the management panel.
*   **Model Configuration**: Define and manage the Gemini models supported by this proxy in the management panel.
*   **Intuitive Management Interface**: Provides a Web UI (`/login` or `/admin`) to view API usage statistics and configure settings.
*   **One-Click Deployment**: Supports quick deployment to the Cloudflare Workers platform via the "Deploy to Cloudflare" button.
*   **GitHub Actions Automatic Deployment**: After forking the repository, enables automatic deployment via GitHub Actions upon code push.

## Deployment

You can choose any of the following methods for deployment:

### Method 1: Quick Deployment

1.  Click the [![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/dreamhartley/gemini-proxy-panel) button above.
2.  Follow the prompts from Cloudflare to complete the deployment process, authorizing access to your GitHub repository.
3.  After deployment, you will need to perform the necessary **Post-Deployment Configuration**.

### Method 2: Automatic Deployment via GitHub Actions(Recommended)

This method is suitable for users who want to manage the code in their own forked repository and automatically update the deployment via `git push`.

1.  **Fork the Repository**:
    *   Click the "Fork" button in the upper right corner of this repository to fork it to your own GitHub account.

2.  **Obtain Cloudflare Information**:
    *   **Get Account ID**:
        *   Log in to the Cloudflare Dashboard.
        *   Find and copy your "Account ID" from the right sidebar on the main page. Alternatively, navigate to "Workers & Pages", and you can find it on the overview page's right side.
    *   **Get API Token**:
        *   In the Cloudflare Dashboard, click your user icon in the top right -> "My Profile" -> "API Tokens".
        *   Click "Create Token".
        *   Find the "Edit Cloudflare Workers" template and click "Use template".
        *   (Optional) You can adjust the permission scope as needed, but the default template permissions are usually sufficient. Ensure it includes at least `Workers Scripts:Edit` and `Workers KV Storage:Edit` permissions for the `Account` resource.
        *   Select your account resources and zone resources (usually keep the default "Include" -> "All zones").
        *   Click "Continue to summary", then click "Create Token".
        *   **Immediately copy the generated API Token**. This token is shown only once, so store it securely.

3.  **Set Secrets in Your GitHub Repository**:
    *   Go to the GitHub page of **your forked repository**.
    *   Click "Settings" -> "Secrets and variables" -> "Actions".
    *   Click the "New repository secret" button and add the following two secrets:
        *   `CF_ACCOUNT_ID`: Paste your Cloudflare Account ID obtained earlier.
        *   `CF_API_TOKEN`: Paste the Cloudflare API Token you created and copied earlier.

4.  **(Optional) Configure PAT for Automatic Action Updates**:
    *   If you want GitHub Actions to be able to automatically update workflow files during runtime (e.g., if the upstream repository updates `.github/workflows/deploy.yml`, your fork could potentially pull these updates automatically via some mechanism), or perform other actions requiring write permissions to the repository, you might need to configure a Personal Access Token (PAT).
    *   **Create PAT**:
        *   Go to your GitHub "Settings" -> "Developer settings" -> "Personal access tokens" -> "Tokens (classic)".
        *   Click "Generate new token" -> "Generate new token (classic)".
        *   Give the token a descriptive name, e.g., `WORKFLOW_UPDATE_PAT`.
        *   Set an expiration date.
        *   Under "Select scopes", check the `workflow` scope (Update GitHub Action workflows).
        *   Click "Generate token".
        *   **Immediately copy the generated PAT** and store it securely.
    *   **Set PAT Secret**:
        *   Return to your forked repository's "Settings" -> "Secrets and variables" -> "Actions".
        *   Click "New repository secret" and add the following secret:
            *   `PAT`: Paste the GitHub Personal Access Token you just created and copied.

5.  **Trigger Deployment**:
    *   When you push code changes to the `main` branch of your forked repository, the GitHub Actions deployment workflow will automatically trigger.
    *   You can also go to the "Actions" tab of your repository, find the "Deploy to Cloudflare Workers" workflow, and manually trigger a run.

6.  **Complete Post-Deployment Configuration**:
    *   After the first successful deployment via GitHub Actions, you still need to follow the **Post-Deployment Configuration** steps below to create and bind KV Namespaces and set environment variables in the Cloudflare Dashboard.

### Method 3: Manual Deployment (Using Wrangler)

1.  Clone this repository:
    ```bash
    git clone https://github.com/dreamhartley/gemini-proxy-panel.git
    cd gemini-proxy-panel
    ```
2.  Install dependencies:
    ```bash
    npm install
    ```
3.  Log in to Wrangler:
    ```bash
    npx wrangler login
    ```
4.  (Optional) Modify the `name` (Worker name) and `account_id` in the `wrangler.toml` file. If deploying via GitHub Actions, `account_id` will be read from the secret.
5.  Deploy:
    ```bash
    npx wrangler deploy
    ```

### Post-Deployment Configuration (Critical Steps)

**Regardless of the deployment method used**, after the initial successful deployment, you need to perform the following configurations in the Cloudflare Dashboard:

1.  **Create KV Namespaces**:
    *   In the Cloudflare Dashboard, navigate to "Workers & Pages" -> "KV".
    *   Create two KV Namespaces:
        *   `GEMINI_KEYS_KV`
        *   `WORKER_CONFIG_KV`
    *   Note down their Namespace IDs (though you usually select by name when binding).

2.  **Bind KV Namespaces to Worker**:
    *   Navigate to your deployed Worker (under "Workers & Pages").
    *   Go to the Worker's "Settings" -> "Variables".
    *   In the "KV Namespace Bindings" section, click "Edit variables", then add two bindings:
        *   Variable name: `GEMINI_KEYS_KV`, KV Namespace: Select the `GEMINI_KEYS_KV` you just created.
        *   Variable name: `WORKER_CONFIG_KV`, KV Namespace: Select the `WORKER_CONFIG_KV` you just created.
    *   Click "Save".

3.  **Set Environment Variables**:
    *   On the same Worker's "Settings" -> "Variables" page.
    *   In the "Environment Variables" section, click "Edit variables", then add the following variables (it's recommended to click "Encrypt" next to the value for added security):
        *   `ADMIN_PASSWORD`: Set a secure password to log in to the management panel.
        *   `SESSION_SECRET_KEY`: Set a long and random string for session management. You can use a password generator to create a strong random string (e.g., at least 32 characters).
    *   Click "Save".

4.  **Redeploy (If Necessary)**:
    *   Cloudflare usually applies binding and environment variable changes automatically. However, if the Worker doesn't immediately pick up the latest bindings and variables, you might need to manually trigger a new deployment (e.g., via Wrangler `npx wrangler deploy`, by editing code in the Cloudflare dashboard and clicking "Deploy", or by pushing again/manually triggering the GitHub Action).

## Usage

### Management Panel

1.  Access the `/login` or `/admin` path of your Worker URL (e.g., `https://your-worker-name.your-subdomain.workers.dev/login`).
2.  Log in using the `ADMIN_PASSWORD` you set.
3.  In the management panel, you can:
    *   Add and manage your Gemini API keys.
    *   Add and manage API keys used to access this Worker proxy (Worker API Keys).
    *   Set global quotas for Pro and Flash series models.
    *   View usage statistics for each Gemini API key.
    *   Configure supported Gemini models.

### API Proxy

1.  Point the API endpoint of your application (originally configured to call the OpenAI API) to your deployed Worker URL (e.g., `https://your-worker-name.your-subdomain.workers.dev/v1`).
2.  Ensure that your application includes valid authentication information when sending requests. This is usually done by carrying the "Worker API Key" configured in the management panel in the `Authorization` request header:
    ```
    Authorization: Bearer <your_worker_api_key>
    ```
3.  Send requests compatible with the OpenAI Chat Completions API. The Worker will convert them into Gemini API requests and return the formatted response.

## Configuration Overview

*   **KV Namespaces (Must Be Bound)**:
    *   `GEMINI_KEYS_KV`: Stores Gemini API keys and their usage.
    *   `WORKER_CONFIG_KV`: Stores Worker configurations, such as Worker API keys, supported models, etc.
*   **Environment Variables (Must Be Set)**:
    *   `ADMIN_PASSWORD`: The login password for the management panel.
    *   `SESSION_SECRET_KEY`: The key used to protect user session security.
*   **GitHub Actions Secrets (Required for Automatic Deployment)**:
    *   `CF_ACCOUNT_ID`: Your Cloudflare Account ID.
    *   `CF_API_TOKEN`: Cloudflare API Token used for deploying the Worker.
    *   `PAT` (Optional): Personal Access Token for GitHub Actions workflow updates or other operations requiring repository write permissions.
