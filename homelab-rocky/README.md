# Rocky SELinux Exercise for Cybersecurity Professionals

This lab demonstrates how SELinux (Security-Enhanced Linux) provides Mandatory Access Control (MAC) to restrict services, even if they have been compromised or are running as root.

## Core Concepts for Cybersecurity Professionals

### 1. MAC vs DAC
- **Discretionary Access Control (DAC):** The traditional Linux permissions (`rwx`). If a process runs as `root`, it can access almost anything.
- **Mandatory Access Control (MAC):** SELinux adds a second layer. Even if a process is `root`, it can only access files and ports that its **context** allows.

### 2. Type Enforcement
SELinux uses "Types" to label everything:
- **Processes** get a domain (e.g., `httpd_t`).
- **Files** get a type (e.g., `httpd_sys_content_t`).
- **Ports** get a type (e.g., `http_port_t`).

The policy says: "Processes of type `httpd_t` can read files of type `httpd_sys_content_t` and bind to ports of type `http_port_t`."

## The Exercise Scenario
We configure an Apache web server to:
1. Listen on a non-standard port: **8081**
2. Serve content from a non-standard directory: **`/opt/myweb`**

### Steps to Reproduce the "Compromised" or "Blocked" State
1. Run the setup playbook:
   ```bash
   ansible-playbook selinux_lab.yml
   ```
2. Observe that Apache fails to start or fails to serve content.

### Troubleshooting (The Cyber Pro Way)

#### A. Checking File Contexts
Check if the web directory has the correct label:
```bash
ls -Z /opt/myweb
```
You will likely see `default_t` or `usr_t`. Apache (`httpd_t`) cannot read these.

#### B. Checking Port Contexts
Check which ports are allowed for HTTP:
```bash
semanage port -l | grep http_port_t
```
You will see that 8081 is NOT in the list.

#### C. Investigating the Audit Log
SELinux denials are logged in `/var/log/audit/audit.log`. Look for `type=AVC`:
```bash
grep "AVC" /var/log/audit/audit.log
```

#### D. Using Sealert (Human Readable)
If `setroubleshoot-server` is installed, use `sealert` to get an explanation and suggested fix:
```bash
sealert -a /var/log/audit/audit.log
```

## Remediation (Applying the Fix)
Instead of disabling SELinux (`setenforce 0`), we apply the "Least Privilege" principle:

1. **Allow the port:**
   ```bash
   semanage port -a -t http_port_t -p tcp 8081
   ```
2. **Label the directory:**
   ```bash
   semanage fcontext -a -t httpd_sys_content_t "/opt/myweb(/.*)?"
   restorecon -Rv /opt/myweb
   ```
3. **Run the remediation playbook:**
   ```bash
   ansible-playbook selinux_remediation.yml
   ```

## Advanced: Generating Custom Policies
Sometimes, standard types aren't enough. In these cases, we use `audit2allow` to generate a custom policy module (`.pp`). This is useful for proprietary or legacy software.

1. Find the denial: `grep "AVC" /var/log/audit/audit.log`
2. Generate a module:
   ```bash
   grep "AVC" /var/log/audit/audit.log | audit2allow -M my_custom_service
   ```
3. Load the module:
   ```bash
   semodule -i my_custom_service.pp
   ```
   *Note: Use this with caution. Always audit the generated `.te` file first.*

## Why This Matters in Cybersecurity
By using SELinux, even if an attacker finds a Zero-Day vulnerability in Apache and gains a shell as the `apache` user (or even `root`), they are restricted:
- They cannot read `/etc/shadow` (because Apache's context doesn't allow it).
- They cannot start a reverse shell on a random port.
- They cannot access other users' home directories.
- They are "caged" within the `httpd_t` domain.
