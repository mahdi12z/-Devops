

## GitLab Troubleshooting: Real-World Issues and Fixes"


SSH keys added via the GitLab UI were not being applied on the host, causing errors like:

```
Permission denied (publickey)
```

when using git clone, git push, etc.

Internal webhooks targeting localhost or internal services were being blocked.

The gitlab-shell component wasn’t logging, making it harder to debug SSH access.

Internal SSH command execution failed or behaved incorrectly in shell environments.

```
# Enable automatic management of authorized_keys for SSH access
gitlab_rails['manage_authorized_keys'] = true

# Use interactive login shell for internal SSH execution
gitlab_rails['gitlab_shell']['ssh_command'] = '/bin/bash --login -c'

# Increase logging for GitLab Shell for better debugging
gitlab_rails['gitlab_shell']['log_level'] = 'info'

# Allow local URLs for webhooks (e.g. http://localhost:9000)
gitlab_rails['webhook_local_requests_allowed'] = true

```

```
sudo gitlab-ctl reconfigure
```

---

## Enabling Fast SSH Key Lookup from GitLab Database Using sshd Match User Configuration
## Objective:

To configure GitLab so that SSH keys are retrieved **directly from the GitLab database**, instead of relying on the static `authorized_keys` file. This ensures immediate access for users after adding their SSH key in the GitLab UI, without requiring manual syncing or `reconfigure`.

## Problem:

In some setups, when users add their SSH key through the GitLab web UI, the key **doesn't get synced** to the server's `authorized_keys` file automatically. This leads to access errors such as:

```
`Permission denied (publickey)`
```


This is especially problematic in environments with many users or automation (e.g., CI/CD), where real-time access is expected.

## Solution: Use `gitlab-shell-authorized-keys-check` with `sshd Match User`

GitLab provides a command-line tool called `gitlab-shell-authorized-keys-check` that can query SSH keys **directly from the GitLab database**.

By integrating this tool with the `sshd` configuration, we can instruct OpenSSH to validate user keys in real time.

## Configuration Steps

###  1-Edit the SSH daemon configuration:

Open `/etc/ssh/sshd_config`:
```
sudo vi /etc/ssh/sshd_config
```
## 2-Add the following block before any `Match all` line:
```
Match User git
  AuthorizedKeysCommand /opt/gitlab/embedded/service/gitlab-shell/bin/gitlab-shell-authorized-keys-check git %u %k
  AuthorizedKeysCommandUser git

Match all
```

## 3-Verify the key-check binary exists:

```
ls -l /opt/gitlab/embedded/service/gitlab-shell/bin/gitlab-shell-authorized-keys-check
```
## 4-Restart SSH service:
```
sudo systemctl restart sshd
```

5-Test the SSH key access:
```
ssh -T git@<ip>
```