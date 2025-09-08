# SJSU Ansible Webserver (Port 8080)

This project configures two VMs (VM1 and VM2) and uses Ansible to deploy a webserver on port **8080**. The landing page displays a host-specific message:

> **Hello World from SJSU-X** (where X is 1 for VM1 and 2 for VM2).

## Prerequisites

- Two Ubuntu/Debian-based VMs reachable over SSH (adjust instructions for RHEL-family as needed).
- On your **control machine** (your laptop or a jump box):
  - Python 3.8+
  - Ansible 2.12+
  - SSH key-based auth to VM1 and VM2 (recommended).

## Quick Start

1. Edit `inventory.ini` and set:
   - `ansible_host` for `vm1` and `vm2`
   - `ansible_user` (e.g., `ubuntu` for cloud images)
   - Leave `web_id=1` for `vm1` and `web_id=2` for `vm2`

2. (Optional) Test connectivity:
   ```bash
   ansible -i inventory.ini web -m ping
   ```

3. **Deploy** webservers:
   ```bash
   ansible-playbook -i inventory.ini site.yml --tags deploy
   ```

4. Verify in a browser:
   - `http://<VM1_IP>:8080` → **Hello World from SJSU-1**
   - `http://<VM2_IP>:8080` → **Hello World from SJSU-2**

   Or using curl:
   ```bash
   curl -s http://<VM1_IP>:8080 | grep "SJSU-1"
   curl -s http://<VM2_IP>:8080 | grep "SJSU-2"
   ```

5. **Un-deploy** (tear down) when finished:
   ```bash
   ansible-playbook -i inventory.ini site.yml --tags undeploy
   ```

## Files

- `ansible.cfg` – local Ansible config
- `inventory.ini` – defines `vm1` and `vm2` with `web_id`
- `site.yml` – contains two plays:
  - Deploy role `web` (tag: `deploy`)
  - Undeploy tasks (tag: `undeploy`)
- `roles/web/` – role implementing nginx setup on port 8080
  - `tasks/main.yml` – install and configure nginx
  - `handlers/main.yml` – reload handler
  - `templates/index.html.j2` – hello page using `web_id`
  - `templates/nginx_8080.conf.j2` – server block

## Notes

- This role removes the default nginx site to avoid port conflicts.
- If using a firewall (e.g., UFW), allow port 8080:
  ```bash
  sudo ufw allow 8080/tcp
  ```

## Troubleshooting

- `UNREACHABLE!` → Check `ansible_host`, SSH connectivity, and user in `inventory.ini`.
- `nginx -t` fails → Fix syntax or conflicting ports; then re-run deploy.
- SELinux (RHEL): set proper contexts or disable for test.