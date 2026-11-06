# kwtSMS for Nagios and Icinga

Send Nagios Core and Icinga 2 alerts by SMS through [kwtSMS](https://www.kwtsms.com), the SMS gateway for Kuwait and
the Gulf. Uses [kwtsms-cli](https://github.com/kwtsms/kwtsms-cli), one file with no dependencies.

## Requirements

- Nagios Core 4, or Icinga 2.
- The monitoring server can reach `https://www.kwtsms.com`.
- A kwtSMS account with API access. [Sign up free](https://www.kwtsms.com/signup/).

## 1. Install kwtsms-cli

```sh
sudo curl -Lo /usr/local/bin/kwtsms-cli https://github.com/kwtsms/kwtsms-cli/releases/latest/download/kwtsms-cli-linux-x64
sudo chmod 755 /usr/local/bin/kwtsms-cli
```

ARM servers: use `kwtsms-cli-linux-arm64` or `kwtsms-cli-linux-armv7`.

## 2. Store the API login for the monitoring user

```sh
sudo mkdir -p /etc/kwtsms
sudo tee /etc/kwtsms/kwtsms-cli.toml >/dev/null <<'CONF'
username = "your_api_username"
password = "your_api_password"
sender   = "KWT-SMS"
CONF
sudo chown nagios:nagios /etc/kwtsms/kwtsms-cli.toml   # Icinga 2: icinga:icinga (Debian: nagios:nagios)
sudo chmod 600 /etc/kwtsms/kwtsms-cli.toml
```

Use the API username and password of your kwtSMS account, not your mobile number. The password stays in this file,
never on a command line where other users could see it.

Check it as the monitoring user, in test mode (queued on kwtSMS, not delivered, no credits used):

```sh
sudo -u nagios kwtsms-cli --config /etc/kwtsms/kwtsms-cli.toml send --test --to 965XXXXXXXX --message "kwtSMS test"
```

## 3a. Nagios Core

1. Copy [`kwtsms.cfg`](kwtsms.cfg) to `/usr/local/nagios/etc/objects/` and add
   `cfg_file=/usr/local/nagios/etc/objects/kwtsms.cfg` to `nagios.cfg`.
2. For each contact who should get SMS: put the number in international format in `pager`, and add
   `notify-host-by-kwtsms` and `notify-service-by-kwtsms` to its notification commands (example in the file).
3. `nagios -v /usr/local/nagios/etc/nagios.cfg`, then reload Nagios.

Keep `illegal_macro_output_chars` in `nagios.cfg` at its default so plugin output cannot break the command line.

## 3b. Icinga 2

1. Copy [`kwtsms-icinga2.conf`](kwtsms-icinga2.conf) to `/etc/icinga2/conf.d/kwtsms.conf`.
2. Give each user a `pager` number and apply the notifications (example in the file).
3. `icinga2 daemon -C`, then `systemctl reload icinga2`.

## Keep alert storms under control

Every SMS uses credits. Send only critical and down states by SMS, use notification intervals and escalations
(e-mail first, SMS when the problem stays), and keep messages short: one SMS page is 160 English characters or 70
Arabic characters. Watch your balance with `kwtsms-cli balance`; at zero, sending stops with `ERR010`.

## Troubleshooting

Nagios logs a failed notification command with its exit code in `nagios.log`; Icinga 2 in its main log. Run the same
command by hand as the monitoring user (step 2) to see the kwtSMS error and its fix, for example `ERR003` (wrong API
login), `ERR010` (zero balance), `ERR024` (server IP not in your kwtSMS API IP whitelist).

## Support

Help: https://www.kwtsms.com/support.html | FAQ: https://www.kwtsms.com/faq/ | support@kwtsms.com

## License

MIT, (c) kwtSMS
