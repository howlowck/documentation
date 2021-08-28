---
title: "Deploy to Scaleway"
metaTitle: "Deploy to Scaleway"
metaDescription: "Step-by-step guide for deploying OpenReplay on Scaleway Elements."
---

OpenReplay stack can be installed on a single instance and Scaleway Elements is an ideal candidate. Here's how to do it.

## Launch an instance

1. Go to Scaleway Dashboard
2. Navigate to 'Compute > Instances' then click 'Create an instance'
3. Choose your preferred Availability Zone
4. Select an Image. For this guide, we'll be using *Ubuntu Server 20.04 Focal Fossa*
5. Choose your instance type. The minimum specs are `2 vCPUs, 8 GB of RAM, 50 GB of storage`. So, we recommend the `DEV1-L` (or an equivalent) for a low/moderate volume. If you're expecting high traffic, you should scale from here.
6. Add Volumes: Set the size to at least 50 GB (whether local or block storage)
7. Give your instance a sweet name (i.e. openreplay)
8. SSH Keys: Make sure you already have a key associated with your project so you can connect to your instance
9. Click 'Create a new instance'

> **Note:** The SMTP ports (25, 465, 587) are blocked by default by Scaleway. Your OpenReplay instance won't be able to send emails unless you enable SMTP from your security group configuration. To do so, check this [quick tutorial](https://www.scaleway.com/en/faq/why-can-i-not-send-any-email/).

## Deploy OpenReplay

1. Make sure your instance is `Started` then connect to it:

```bash
## From your terminal
SSH_KEY=~/Downloads/openreplay-key.pem #! wherever you've saved the SSH key
INSTANCE_IP=REPLACE_WITH_INSTANCE_PUBLIC_IP
chmod 400 $SSH_KEY
ssh -i $SSH_KEY root@$INSTANCE_IP
```

2. Install OpenReplay:

```shellsession
git clone https://github.com/openreplay/openreplay.git
cd openreplay/scripts/helm && bash install.sh
```
> **Note:** You'll be prompted to provide the domain on which OpenReplay will be running (e.g. openreplay.mycompany.com). This is required to continue the installation.

## Configure TLS/SSL

OpenReplay deals with sensitive user data and therefore requires HTTPS to run. This is mandatory, otherwise the tracker simply wouldn't start recording. Same thing for the dashboard, without HTTPS you won't be able to replay user sessions.

You must therefore bring (or generate) your own SSL certificate.

First, go to 'Network' > 'DNS' (or your other DNS service provider) and create an `A Record`. Use the domain you previously provided during the installation step and point it to the instance using its public IP (can be found in 'Compute' > 'Instances').

Open the `vars.yaml` file with the command `vi openreplay/scripts/helm/vars.yaml` then substitute:
- `domain_name`: this is where OpenReplay will be accessible (i.e. openreplay.mycompany.com)
- `nginx_ssl_cert_file_path`: the path to you .cert file (i.e. /root/openreplay/my-cert.crt)
- `nginx_ssl_key_file_path`: the path to your .pem file (i.e. /root/openreplay/my-key.pem)

> **Note:** If you don't have a certificate, generate one for your subdomain (the one provided during installation) using Let's Encrypt. Connect to OpenReplay instance, run `helm uninstall -n nginx-ingress nginx-ingress` then execute `bash openreplay/scripts/certbot.sh` and follow the steps.

Restart OpenReplay NGINX (and choose whether to enable the default HTTP to HTTPS redirection using the `NGINX_REDIRECT_HTTPS` variable):

```bash
cd openreplay/scripts/helm
NGINX_REDIRECT_HTTPS=1 ./install.sh --app nginx
```

You're all set now, OpenReplay should be accessible on your subdomain. You can create an account by visiting the `/signup` page (i.e. openreplay.mycompany.com/signup).

## Troubleshooting

If you encounter any issues, connect to our [Slack](https://slack.openreplay.com) and get help from our community.