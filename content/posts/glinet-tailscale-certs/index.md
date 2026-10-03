+++
title = 'Set up auto-renewing HTTPS certificates on a GLiNet KVM'
date = 2026-10-02T20:21:30-04:00
draft = false
tags = ["tailscale", "glinet", "certificates"]
+++

I'm quite fond of the small little IP KVMs that have become more and more available lately.

My favorites are the JetKVM and the GliNet Comet series.

I have 2 of the Comets that I use internally, and I mostly access them via their Tailscale ts.net names.

I _could_ do I what I usually do and put them behind a reverse proxy, but there's one of these that I keep in my work bag and take with me onto client sites, so there's no reverse proxy in that scenario.

I could just ignore the certificate error, but where's the fun in that when we can fix it?

1. Activate and log into Tailscale on your device.
2. Ensure that HTTPS is enabled for your TailNet. This can be done [here.](https://console.tailscale.com/admin/settings/general)
3. SSH into the GliNet KVM.
4. Update Tailscale, you heathen.
    ```bash
    tailscale update
    ```
5. Once Tailscale is updated, lets create the renewal script.
    ```bash
    vi /etc/tailscale-cert-renew.sh
    ```

    And the contents of the script:

    ```sh
    #!/bin/sh

    set -eu

    HOSTNAME="your-kvm.your-tailnet.ts.net"
    CERT_DIR="/etc/kvmd/user/ssl"
    TMP_DIR="/tmp/tailscale-cert"
    RENEWAL_WINDOW=1209600 # 14 days

    # Exit if the existing certificate is valid for more than 14 days.
    if openssl x509 \
        -checkend "$RENEWAL_WINDOW" \
        -noout \
        -in "$CERT_DIR/server.crt" >/dev/null 2>&1; then
        exit 0
    fi

    echo "Certificate expires within 14 days. Renewing..."

    rm -rf "$TMP_DIR"
    mkdir -p "$TMP_DIR"

    CERT="$TMP_DIR/server.crt"
    KEY="$TMP_DIR/server.key"

    tailscale cert \
        --cert-file="$CERT" \
        --key-file="$KEY" \
        "$HOSTNAME"

    # Make sure we actually received a valid certificate.
    openssl x509 \
        -in "$CERT" \
        -noout \
        -subject \
        -issuer \
        -dates

    # Replace the existing certificate and key.
    cp "$CERT" "$CERT_DIR/server.crt"
    cp "$KEY" "$CERT_DIR/server.key"

    rm -rf "$TMP_DIR"

    # Restart the KVM's nginx instance.
    /etc/init.d/S99kvmd-nginx restart

    echo "Tailscale certificate renewed and nginx restarted."
    ```

6. Make the script executable.
    ```bash
    chmod 700 /etc/tailscale-cert-renew.sh
    ```
7. Run the script once to test it.
    ```bash
    /etc/tailscale-cert-renew.sh
    ```

    You should be able to reload the GLiNet web interface and see that you have a valid LetsEncrypt certificate for your Tailscale MagicDNS name.

    {{% callout aside %}}
    Make sure you're accessing the device at `your-kvm.yourtailnet.ts.net` and not it's IP address, or the certificate won't validate.
    {{% /callout %}}

8. Now, lets create a cron job to run the script automatically. The GliNet KVMs already run a cron daemon (at least the ones that I've used do) - so you should be able to directly edit the crontab.
    ```bash
    crontab -e
    ```

    To run the script once a day at 0300:

    ```
    0 3 * * * /etc/tailscale-cert-renew.sh
    ```

You should now have a GliNet KVM that automatically renews your Tailscale certificates when needed. :)
