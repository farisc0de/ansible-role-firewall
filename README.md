# Ansible Role: Firewall

An Ansible role to manage firewall rules across different Linux distributions. This role automatically detects the target system and applies the appropriate firewall configuration using:
- `firewalld` for RedHat-based systems (RHEL, CentOS, Fedora)
- `iptables` for Debian-based systems (Debian, Ubuntu)

## Requirements

- Ansible 2.9 or higher
- Root privileges on target hosts
- For RedHat systems: firewalld
- For Debian systems: iptables and iptables-persistent

## Role Variables

All variables are defined in `defaults/main.yml`. Here are the key variables you can customize:

```yaml
# TCP ports to allow (default common ports)
firewall_allowed_tcp_ports:
  - "80"   # HTTP
  - "443"  # HTTPS
  - "22"   # SSH

# UDP ports to allow (empty by default)
firewall_allowed_udp_ports: []

# Custom rules with source IP restrictions
firewall_custom_rules: []  # Format: { port: PORT, protocol: tcp/udp, source: SOURCE_IP }

# Whether to save rules after changes
firewall_save_rules: true

# Default policies
firewall_default_input_policy: "DROP"    # Default: DROP all incoming traffic
firewall_default_forward_policy: "DROP"   # Default: DROP all forwarded traffic
firewall_default_output_policy: "ACCEPT"  # Default: ACCEPT all outgoing traffic
```

### Custom Rules Example

You can define custom rules with source IP restrictions:

```yaml
firewall_custom_rules:
  - port: "8080"
    protocol: tcp
    source: "192.168.1.0/24"  # Optional: restrict to specific source IP/network
  - port: "53"
    protocol: udp
    source: "10.0.0.0/8"
```

## Dependencies

None.

## Example Playbook

Basic usage:

```yaml
- hosts: servers
  become: true
  roles:
    - role: ansible-role-firewall
```

Advanced usage with custom configuration:

```yaml
- hosts: servers
  become: true
  roles:
    - role: ansible-role-firewall
      vars:
        firewall_allowed_tcp_ports:
          - "80"    # HTTP
          - "443"   # HTTPS
          - "22"    # SSH
          - "3306"  # MySQL
        firewall_allowed_udp_ports:
          - "53"    # DNS
        firewall_custom_rules:
          - port: "8080"
            protocol: tcp
            source: "192.168.1.0/24"
```

## Role Behavior

### RedHat-based Systems
- Uses `firewalld` for firewall management
- Installs and enables firewalld service
- Configures permanent rules
- Automatically restarts firewalld when rules change

### Debian-based Systems
- Uses `iptables` for firewall management
- Installs iptables and iptables-persistent
- Saves rules using netfilter-persistent
- Configures rules with proper state tracking

## Tags

The role provides several tags for selective execution:

- `firewall`: All firewall-related tasks
- `firewall_install`: Installation tasks only
- `firewall_config`: Configuration tasks only

## Security Notes

- The role defaults to a secure configuration with DROP policies for INPUT and FORWARD chains
- All incoming traffic is denied by default unless explicitly allowed
- Established connections are automatically allowed
- Loopback interface traffic is allowed
- Output traffic is allowed by default but can be restricted by changing `firewall_default_output_policy`

## License

MIT

## Author Information

Created and maintained by Faris AL-Otaibi.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a new Pull Request
