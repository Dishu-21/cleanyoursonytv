# Sony TV shows "No internet" after waking until Wi-Fi is toggled

## What the fork changes
- When Home comes to the front and the default network is linked but not validated, it asks Android to re-run
  its connectivity check (`reportNetworkConnectivity`). At most once per 30 seconds, no extra permission.
- The Wi-Fi pill says "No internet" in that state instead of showing the network name.
This is a mitigation. It cannot fix a cause outside the launcher, so if the pill keeps saying "No internet",
run the checks below with the TV in the broken state.

## Checks to run while it is broken (from the repo folder, TV_HOST set)
```
tools/venv/bin/python -I tools/tv.py 'dumpsys connectivity | grep -i -E "validated|NetworkAgentInfo|captive" | head -20'
tools/venv/bin/python -I tools/tv.py 'settings get global private_dns_mode; settings get global private_dns_specifier'
tools/venv/bin/python -I tools/tv.py 'settings get global captive_portal_mode'
tools/venv/bin/python -I tools/tv.py 'pm list packages -d'
tools/venv/bin/python -I tools/tv.py 'dumpsys wifi | grep -i -E "power|sleep|suspend" | head -20'
```

## Likely causes, most to least likely
1. Private DNS pointed at an ad-blocking resolver (README FAQ). If the TV wakes before the resolver is reachable,
   validation fails and sticks. Test: `settings put global private_dns_mode opportunistic`, then sleep and wake the TV.
2. A disabled Sony network package. Sony's `homenetwork` package holds network standby settings; keep it enabled.
   Compare `pm list packages -d` with your debloat list and re-enable suspects with `pm enable --user 0 <package>`.
3. Sony's network standby or Wi-Fi power saving: on the TV, Settings > Network & Internet and Settings > Device
   Preferences > Power (names vary by model), look for network standby or remote start and try turning it on or off.
4. Router side: a Wi-Fi 6 or band-steering router can drop the TV's session in standby. Try a fixed 5 GHz band
   for the TV or a DHCP reservation.
