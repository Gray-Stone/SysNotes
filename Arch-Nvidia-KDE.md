
Apart from the default arch wiki nvidia setup guide, There are few extra items in the following article for grub.

https://linuxiac.com/nvidia-with-wayland-on-arch-setup-guide/



## Error on Plasma shell and Kwin after long sleep

One option is to just restart the Kwin without logging out


https://askubuntu.com/questions/481329/can-i-restart-the-kde-plasma-desktop-without-logging-out



```
kquitapp5 plasmashell
kstart5 plasmashell
```
Stop and re-start plasmashell. 

However vscodiem seems to be acting strange after this. not sure why
