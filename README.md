# Solace Queue Egress Management Pipeline

This Jenkins pipeline automates enabling or disabling queue egress in
Solace Message VPN queues using configuration files stored in a Git
repository.

------------------------------------------------------------------------

## Overview

This pipeline performs queue egress operations using two configuration
files:

-   AEM_Queue.yaml
-   AEM_Config.yaml

It uses the SEMP v2 API to update queue configuration in Solace.

------------------------------------------------------------------------

## Pipeline Parameters

  -----------------------------------------------------------------------
  Parameter                   Type          Description
  --------------------------- ------------- -----------------------------
  ACTION                      Choice        Select `Egress Enable` or
                                            `Egress Disable`

  BITBUCKET_REPO_URL          String        Repository containing
                                            configuration files

  BRANCH_NAME                 String        Git branch to checkout

  GIT_CREDENTIALS_ID          String        Jenkins credentials ID for
                                            repository access
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## Repository Structure

Your configuration repository must contain:

repo-root/ │ ├── AEM_Queue.yaml └── AEM_Config.yaml

------------------------------------------------------------------------

## Configuration File Formats

### AEM_Queue.yaml

Defines queues grouped by VPN.

Example:

``` yaml
AEM_Queue:
  VPN1:
    - queue/test1
    - queue/test2
  VPN2:
    - queue/sample1
```

------------------------------------------------------------------------

### AEM_Config.yaml

Defines connection details for each VPN.

Example:

``` yaml
AEM_Config:
  VPN1:
    Host: solace-host.example.com
    UserId: admin
    UserPass: password

  VPN2:
    Host: solace-host2.example.com
    UserId: admin
    UserPass: password
```

------------------------------------------------------------------------

## Pipeline Stages

### 1. Validate Input

Validates required parameters: - Repository URL - Branch name

### 2. Checkout Configuration Files

-   Clones repository into workspace
-   Verifies required YAML files exist

### 3. Parse Configuration Files

-   Reads YAML files
-   Converts them into JSON for processing
-   Displays queue summary

### 4. Process Queue Egress Operations

For each VPN and queue: - Validates VPN configuration - Builds SEMP API
request - Sends PATCH request using curl - Records results

Example API call:

PATCH /SEMP/v2/config/msgVpns/{vpn}/queues/{queue}

Payload:

``` json
{
  "egressEnabled": true
}
```

------------------------------------------------------------------------

## Operation Summary Report

At completion, the pipeline prints:

-   Total queues processed
-   Successful operations
-   Failed operations
-   Skipped operations

Detailed results are stored as:

-   queueConfig.json
-   solaceConfig.json
-   successResults.json
-   failedResults.json
-   skippedResults.json

------------------------------------------------------------------------

## Jenkins Requirements

The Jenkins instance must have:

-   Pipeline plugin
-   Git plugin
-   Pipeline Utility Steps plugin
-   Credentials configured for repository access
-   Network access to Solace SEMP endpoint
-   curl available on the agent

------------------------------------------------------------------------

## Security Notes

Avoid storing credentials directly in YAML files in production.

Recommended approach: - Use Jenkins Credentials - Use secret injection -
Use credential binding

------------------------------------------------------------------------

## Future Improvements

Possible enhancements:

-   Parallel queue processing
-   Retry mechanism
-   Dry-run mode
-   Email/Slack notifications
-   Vault integration
-   Queue validation step
-   API response logging

------------------------------------------------------------------------

Author: ##Suryanath Tripathy
