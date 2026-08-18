# SRX-1 — root authentication

Run this locally in configuration mode before committing the SRX configuration:

    set system root-authentication plain-text-password

Junos will prompt for the root password twice. Do not add the password, an encrypted password hash, or this device's full configuration backup to GitHub.

Then continue with the numbered `.set` files and commit.
