# ACM GRC Ansible Integration Example

An example of the Red Hat Advanced Cluster Management (ACM) Governance, Risk,
and Compliance (GRC) integration with Ansible for a blog post.

The scenario is that we want a Service Now incident to be created when a
TLS (i.e. SSL) certificate is close to expiring based on an ACM GRC policy.

## OpenShift Setup

This creates a self-signed TLS (i.e. SSL) certifcate. This sets the expiration
date to 25 days from now since the GRC policy to be used will alert in 30 days.

```bash
mkdir -p tls
openssl req \
    -new \
    -newkey rsa:4096 \
    -days 25 \
    -nodes \
    -x509 \
    -subj "/C=US/ST=NC/L=Raleigh/O=Example/CN=www.example.com" \
    -keyout tls/tls.key \
    -out tls/tls.crt
```

This creates a namespace called `acm-grc-ansible-example` and creates a simple
Apache web server deployment using the self-signed certificate generated
previously. Note that the secret stores the TLS key in the key `tls.crt`. This
is the default key name that GRC will check.

```bash
oc create ns acm-grc-ansible-example
oc -n acm-grc-ansible-example create secret generic certs \
    --from-file=tls.key=tls/tls.key \
    --from-file=tls.crt=tls/tls.crt 
oc -n acm-grc-ansible-example apply -f openshift/app.yml
```

## Ansible Setup

The `ansible/playbooks/create_ticket.yml` playbook runs locally and creates
a temporary
[Python virtual environment](https://docs.python.org/3/library/venv.html) and
installs the Python dependencies in it that are required for the
[servicenow.servicenow.snow_record](https://docs.ansible.com/ansible/latest/collections/servicenow/servicenow/snow_record_module.html)
Ansible module. The playbook then creates a Service Now incident.

1. Fork this repository.
1. Create `ansible/vaults/secret-vars.yml` as a new
   [Ansible vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
   file with the Ansible variables of `snow_host` (the FQDN of ServiceNow),
   `snow_password`, and `snow_username`.
1. Commit and push the changes to your fork.
1. Configure Ansible Tower to have a job template that utilizes the playbook in
   `ansible/playbooks/create_ticket.yml`.
