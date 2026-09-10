# How to Check and Stop Services

Use these `systemctl` commands on the Jetson.

Check the project services:

```bash
systemctl status \
  mavros.service \
  mavros-stream-monitor.service \
  luxonis-rlds-recorder.service \
  --no-pager
```

List all currently running services:

```bash
systemctl list-units --type=service --state=running
```

## Check Failed Services

List services that have failed:

```bash
systemctl --failed --type=service
```

Check the status of a failed service and view its recent logs:

```bash
systemctl status SERVICE_NAME --no-pager
journalctl -u SERVICE_NAME -b --no-pager
```

Stop the failed service now and disable it so it does not start automatically:

```bash
sudo systemctl disable --now SERVICE_NAME
```

For example:

```bash
sudo systemctl disable --now mavros.service
```

After fixing or stopping the service, clear its failed state and confirm that no services remain failed:

```bash
sudo systemctl reset-failed SERVICE_NAME
systemctl --failed --type=service
```

List services configured to start automatically:

```bash
systemctl list-unit-files --type=service --state=enabled
```

Stop a service temporarily:

```bash
sudo systemctl stop SERVICE_NAME
```

It can start again after reboot or through another dependency.

Disable automatic startup but leave it running now:

```bash
sudo systemctl disable SERVICE_NAME
```

Stop it now and disable future automatic startup:

```bash
sudo systemctl disable --now SERVICE_NAME
```

For example, to disable the recorder and its monitor while leaving MAVROS running:

```bash
sudo systemctl disable --now \
  mavros-stream-monitor.service \
  luxonis-rlds-recorder.service
```

To disable all three project services:

```bash
sudo systemctl disable --now \
  luxonis-rlds-recorder.service \
  mavros-stream-monitor.service \
  mavros.service
```

Confirm:

```bash
systemctl is-active \
  mavros.service \
  mavros-stream-monitor.service \
  luxonis-rlds-recorder.service

systemctl is-enabled \
  mavros.service \
  mavros-stream-monitor.service \
  luxonis-rlds-recorder.service
```

Re-enable them later:

```bash
sudo systemctl enable --now \
  mavros.service \
  mavros-stream-monitor.service \
  luxonis-rlds-recorder.service
```

If a disabled service keeps being started by dependencies, you can prevent all starts with:

```bash
sudo systemctl mask --now SERVICE_NAME
```

Undo that with:

```bash
sudo systemctl unmask SERVICE_NAME
sudo systemctl enable --now SERVICE_NAME
```

Only mask services you recognize; masking core networking, login, or system services can make the Jetson difficult to operate.