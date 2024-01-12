# Workarouns

## Screen going black after 30 seconds

Reference: https://www.reddit.com/r/pop_os/comments/eln8bp/screen_going_black_after_30_seconds/

So for some reason dpms (Energy Star features) is being enabled, and I'm not sure why. Trying to enable dpms with a different timeout doesn't work, it just defaults to 30 seconds. So still a mystery as to what is enabling it and why.

Running the following fixes it.

```sh
xset -dpms
```

If these settings don't seem to 'stick', try adding the xset command to your ~/.profile or whichever file contains commands to be run upon login. 