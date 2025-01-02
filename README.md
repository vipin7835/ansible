# CloudWatch Agent Installation and Configuration

This Ansible playbook automates the installation and configuration of the Amazon CloudWatch Agent on an Ubuntu system.

## Playbook Overview

**File Name:** `cloudwatch-agent1.yml`

**Purpose:**
- Installs the Amazon CloudWatch Agent on Ubuntu.
- Configures the agent to monitor system metrics and logs.
- Ensures the agent runs with the desired configuration.

## Prerequisites

1. **Target Hosts:** Ensure the host group `vipin` is defined in your Ansible inventory file.
2. **Ansible:** Installed on the control machine.
3. **Permissions:** The playbook requires sudo privileges on the target hosts.
4. **CloudWatch Agent Configuration Template:** Ensure the file `cloudwatch-agent-config.json.j2` exists in your Ansible `templates/` directory.

## Playbook Details

### Steps Included

1. **Install Dependencies**
   - Installs `curl` and `unzip`, required for downloading and extracting the CloudWatch Agent package.

2. **Download CloudWatch Agent Package**
   - Fetches the latest CloudWatch Agent package from AWS S3.

3. **Install CloudWatch Agent**
   - Installs the downloaded `.deb` package.

4. **Enable and Start the CloudWatch Agent**
   - Ensures the CloudWatch Agent service is enabled and running.

5. **Create Configuration Directory**
   - Ensures the required directory for the configuration file exists.

6. **Copy Configuration File**
   - Deploys the configuration file from the template `cloudwatch-agent-config.json.j2`.

7. **Apply Configuration**
   - Uses the CloudWatch Agent CLI to apply the configuration file.

8. **Restart CloudWatch Agent**
   - Restarts the agent to apply the new configuration.

9. **Validate Agent Status**
   - Checks and displays the status of the CloudWatch Agent.

### Key Variables

- **`src:`** Path to the CloudWatch Agent configuration template (`cloudwatch-agent-config.json.j2`).
- **`dest:`** Target path on the managed node (`/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json`).

### Tasks Breakdown

- **Dependencies:**
  Ensures required packages are installed.

  ```yaml
  - name: Install dependencies
    apt:
      name:
        - curl
        - unzip
      state: present
      update_cache: yes
  ```

- **Download Package:**
  Downloads the CloudWatch Agent package from AWS.

  ```yaml
  - name: Download CloudWatch Agent package
    get_url:
      url: "https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb"
      dest: /tmp/amazon-cloudwatch-agent.deb
  ```

- **Configuration Application:**
  Applies the specified configuration using the CloudWatch Agent CLI.

  ```yaml
  - name: Apply CloudWatch Agent configuration
    shell: |
      /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
      -a fetch-config \
      -m ec2 \
      -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json \
      -s
  ```

## Usage

1. **Define Inventory:** Ensure `vipin` is part of your inventory file.

   ```ini
   [vipin]
   <your-host-ip> ansible_user=<username>
   ```

2. **Run the Playbook:**

   Execute the playbook using the following command:

   ```bash
   ansible-playbook cloudwatch-agent1.yml
   ```

3. **Verify Status:**
   Check the debug output for the CloudWatch Agent status to ensure it is running correctly.

## Notes

- Modify `cloudwatch-agent-config.json.j2` to customize the CloudWatch Agent's behavior based on your monitoring requirements.
- Ensure the target system has access to the internet to download the package from AWS S3.

## Troubleshooting

- If the playbook fails, verify the target host's connectivity and permissions.
- Check the `agent_status` output for any issues with the CloudWatch Agent.

