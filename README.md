# Debugging `.github/workflows/main.yml`

This workflow is designed to set up a headless Ubuntu desktop environment and install RustDesk in headless mode. Below are the key steps, potential issues, and debugging tips for each section of the workflow:

---

## Key Steps & Debugging Tips

### 1. Install Ubuntu Desktop
```yaml
- name: Install Ubuntu Desktop
  run: sudo apt-get update && sudo apt-get install -y ubuntu-desktop
```
- **Debug Tip:** This step can be time-consuming and requires significant resources. If the step fails, check for out-of-memory errors or lack of disk space in the Actions runner logs.

### 2. Install Dummy Xorg Video Driver
```yaml
- name: Install Dummy Xorg Video Driver
  run: sudo apt-get install -y xserver-xorg-video-dummy
```
- **Debug Tip:** If installation fails, check if the package is available in your Ubuntu version's repositories.

### 3. Install LightDM
```yaml
- name: Install LightDM
  run: sudo apt-get install -y lightdm
```
- **Debug Tip:** LightDM may prompt for configuration. If the workflow hangs, try adding `DEBIAN_FRONTEND=noninteractive` before the command:
  ```yaml
  run: sudo DEBIAN_FRONTEND=noninteractive apt-get install -y lightdm
  ```

### 4. Install dependencies (wget)
```yaml
- name: Install dependencies
  run: sudo apt-get install -y wget
```
- **Debug Tip:** Usually straightforward. If wget is already installed, this will do nothing.

### 5. Download RustDesk 1.2 nightly
```yaml
- name: Download RustDesk 1.2 nightly
  run: |
    wget -O rustdesk.deb https://github.com/rustdesk/rustdesk/releases/download/nightly/rustdesk-1.2.0-nightly-amd64.deb
```
- **Debug Tip:** If the download fails, check if the URL is valid and accessible. If RustDesk changes the nightly release filename or location, update the URL accordingly.

### 6. Install RustDesk
```yaml
- name: Install RustDesk
  run: sudo dpkg -i rustdesk.deb || sudo apt-get install -f -y
```
- **Debug Tip:** The command attempts to fix dependencies if the initial install fails. If RustDesk is not installed after this step, check the logs for dependency problems.

### 7. Enable RustDesk headless mode
```yaml
- name: Enable RustDesk headless mode
  run: sudo rustdesk --option allow-linux-headless Y
```
- **Debug Tip:** If this step fails, check if the `rustdesk` binary is in the system's PATH (`which rustdesk`). If not, you may need to specify the full path (e.g., `/usr/bin/rustdesk`).

### 8. Set RustDesk password
```yaml
- name: Set RustDesk password
  run: sudo rustdesk --password R0und3mu
```
- **Debug Tip:** If the command fails, check for typos or changes to RustDesk's CLI options.

### 9. Get RustDesk ID
```yaml
- name: Get RustDesk ID
  id: rustdesk_id
  run: |
    echo "id=$(sudo rustdesk --get-id)" >> $GITHUB_OUTPUT
```
- **Debug Tip:** Ensure the previous steps succeeded; otherwise, `rustdesk --get-id` may fail or return nothing. Check the Action step logs for the output.

### 10. Display RustDesk Connection Info
```yaml
- name: Display RustDesk Connection Info
  run: |
    echo "RustDesk Headless is enabled."
    echo "ID: ${{ steps.rustdesk_id.outputs.id }}"
    echo "Password: R0und3mu"
    echo "Connect using the RustDesk client with the above ID and password."
```
- **Debug Tip:** If `${{ steps.rustdesk_id.outputs.id }}` is blank, the previous step failed or did not set the output properly. Confirm the syntax and output name.

---

## General Debugging

- **Check Logs:** For each step, expand the "Run" logs in the GitHub Actions UI to see detailed output and errors.
- **Runner Limitations:** GitHub-hosted runners may not support running a full desktop environment in a meaningful way. For headless use, ensure your use case fits the limitations.
- **Persistent State:** Each workflow run starts with a clean environment. Any data stored during one job won't persist to the next unless explicitly saved.
- **Package Versions:** If installation fails, check the version compatibility of each package with `ubuntu-latest`.

---

## Recommendations

- **Add `set -e` to shell scripts** to fail fast when a command fails.
- **Use `DEBIAN_FRONTEND=noninteractive`** for all `apt-get install` steps to avoid interactive prompts.
- **Check RustDesk Documentation** for the latest CLI options and installation instructions.
- **Test Locally** on a fresh Ubuntu VM to validate steps before running in GitHub Actions.

---

## Example of Improved Apt Install Command

```yaml
run: |
  sudo apt-get update
  sudo DEBIAN_FRONTEND=noninteractive apt-get install -y ubuntu-desktop xserver-xorg-video-dummy lightdm wget
```

---

If you encounter a specific error, please provide the error message for more targeted debugging tips.
