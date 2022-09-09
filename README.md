# pi-spotify

1. Use an image like OSMC/Kodi, available from the Raspberry Pi installer.
   The official raspbian image would not output sound over HDMI.
2. Follow the installation instructions for [raspotify](https://github.com/dtcooper/raspotify).
3. Add the raspotify user to the OSMC group to allow HDMI-CEC (cec-utils) access, solving errors related to `/dev/vchiq`:
   `sudo usermod -aG video raspotify`
4. Configure an event handler for raspotify to control the AV receiver following https://nico.is/blog/control-your-av-receiver-with-raspotify/
   The important line being in `/etc/default/raspotify`
   `OPTIONS="--username ••••• --password ••••• --onevent /usr/local/scripts/onevent.sh"`
5. The contents of `/usr/local/scripts/onevent.sh` should look like this:
```bash
#!/bin/bash

echo "Running onevent for $PLAYER_EVENT"

if [ "$PLAYER_EVENT" = "start" ]; then
  echo "turning on receiver"
  echo 'on 5' | /usr/osmc/bin/cec-client -s -d 1
  echo 'tx 2F:82:31:00' | /usr/osmc/bin/cec-client -s -d 1
fi

if [ "$PLAYER_EVENT" = "stop" ]; then
  echo "turning off receiver"
  echo 'standby 5' | /usr/osmc/bin/cec-client -s -d 1
fi

if [ "$PLAYER_EVENT" = "volume_set" -a "$VOLUME" = "65535" ]; then
  echo "increasing receiver volume"
  echo 'tx 15:44:41' | /usr/osmc/bin/cec-client -s -d 1
fi
```

## Useful commands
- Logs: `sudo journalctl -f -u raspotify`
- HDMI devices: `echo 'scan' | /usr/osmc/bin/cec-client -d 1 -s`

## Resources
- CEC guide https://www.cec-o-matic.com/
- Respond to HDMI commands: https://raspberrypi.stackexchange.com/questions/82847/detect-tv-remote-buttons-being-pressed-with-cec-client
- Control spotify-connect via CLI https://github.com/Madh93/Rpotify
