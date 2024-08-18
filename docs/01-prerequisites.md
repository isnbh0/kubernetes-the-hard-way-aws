# Prerequisites

## Amazon Web Services

This tutorial leverages the [Amazon Web Services](https://aws.amazon.com/) to streamline provisioning of the compute infrastructure required to bootstrap a Kubernetes cluster from the ground up. It would cost less than $2 for a 24 hour period that would take to complete this exercise.

> The compute resources required for this tutorial exceed the Amazon Web Services free tier. Make sure that you clean up the resource at the end of the activity to avoid incurring unwanted costs.

ℹ️ For information on setting up one's AWS account, refer to the [Setting up AWS](ref/setting-up-aws.md) guide.

⚠️ IMPORTANT: You should set up a [budget alert](https://us-east-1.console.aws.amazon.com/billing/home?region=us-east-1#/budgets/overview) to avoid unexpected charges.

## Amazon Web Services CLI

### Install the AWS CLI

Follow the AWS CLI [documentation](https://aws.amazon.com/cli/) to install and configure the `aws` command line utility.

Verify the AWS CLI version using:

```
aws --version
```

### Set a Default Compute Region and Zone

This tutorial assumes a default compute region and zone have been configured.

Go ahead and set a default compute region:

```
AWS_REGION=ap-northeast-2

aws configure set default.region $AWS_REGION
```


## Running Commands in Parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. Labs in this tutorial may require running the same commands across multiple compute instances, in those cases consider using tmux and splitting a window into multiple panes with synchronize-panes enabled to speed up the provisioning process.

> The use of tmux is optional and not required to complete this tutorial.

![tmux screenshot](images/tmux-screenshot.png)

> Enable synchronize-panes by pressing `ctrl+b` followed by `shift+:`. Next type `set synchronize-panes on` at the prompt. To disable synchronization: `set synchronize-panes off`.


## Other packages

### jq

Refer this [link](https://jqlang.github.io/jq/download/) and install `jq` based on your OS.


## helm

Refer this [link](https://helm.sh/docs/intro/install/) and install `helm` based on your OS.


## Session Variables

Before proceeding, run the following script to set up functions for managing persistent session variables:

```sh
cat << EOF > setup_session_vars.sh
#!/bin/bash

# File to store session variables
SESSION_FILE=".kubernetes_session_vars"

# Function to set a session variable
set_var() {
    local key=\$1
    local value=\$2
    # Use awk for in-place editing, which is more portable
    awk -v key="\$key" -v value="\$value" '
        \$0 !~ "^"key"=" { print }
        END { print key"="value }
    ' "\$SESSION_FILE" > "\${SESSION_FILE}.tmp" && mv "\${SESSION_FILE}.tmp" "\$SESSION_FILE"
}

# Function to get a session variable
get_var() {
    local key=\$1
    local value
    value=\$(grep "^\${key}=" "\$SESSION_FILE" 2>/dev/null | cut -d '=' -f2)
    if [ -z "\$value" ]; then
        echo "Error: Variable \$key not found in session file." >&2
        return 1
    fi
    echo "\$value"
}

# Function to check if a variable exists and load it if not
ensure_var() {
    local key=\$1
    local value
    value=\$(eval echo \\\$\$key)
    if [ -z "\$value" ]; then
        value=\$(get_var "\$key")
        if [ \$? -eq 0 ]; then
            export "\$key=\$value"
        else
            echo "Error: Variable \$key not found in environment or session file." >&2
            return 1
        fi
    fi
}

# Function to load all session variables
load_vars() {
    if [ -f "\$SESSION_FILE" ]; then
        while IFS='=' read -r key value
        do
            # Skip empty lines and lines without '='
            [ -z "\$key" ] && continue
            [[ "\$key" != *"="* ]] && continue
            
            # Remove any leading/trailing whitespace from key and value
            key=\$(echo "\$key" | awk '{\$1=\$1};1')
            value=\$(echo "\$value" | awk '{\$1=\$1};1')
            
            # Export the variable only if both key and value are non-empty
            [ -n "\$key" ] && [ -n "\$value" ] && export "\$key=\$value"
        done < "\$SESSION_FILE"
    fi
}

# Create session file if it doesn't exist
rm -f "\$SESSION_FILE"
touch "\$SESSION_FILE"

# Load existing variables
load_vars

echo "Session variable functions have been set up. You can now use set_var, get_var, and ensure_var in your scripts."
EOF

chmod +x setup_session_vars.sh
source ./setup_session_vars.sh
```

---

Next: [Installing the Client Tools](02-client-tools.md)
