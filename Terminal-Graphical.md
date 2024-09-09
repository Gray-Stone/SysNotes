Display graphical stuff in terminal 

## Methods 

## true imaging protocol 

These are three protocol that give actual images out, with pixel to pixel control
* sixel
* iterm
* kitty

`img2sixel` is a classic tool for this.

Konsolo 22 include the sixel support for it now.

## Half blocks

Another common technique is to display the half blocks character with colors 
```
▄
```
This is basically a large sized pixel, which kinda work on displaying images.

## Frame buffer

The super old way of displaying a image. 

A old tool called FBI can display image by directly write to Frame buffer. This also require the user have the permission to write to frame buffer (being in group video)

A newer version called FIM is made, using the same trick.
