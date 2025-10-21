

```
sudo apt install fctix5 fctix5-rime
```

A reboot might be needed after getting these setup

## KDE integrated settings menu 

kde-config-fcitx5

for arch, it's `fcitx5-configtool` package that made it show up in kde menu.


## Enable 

Add rime via the fctix5 GUI 

## Config files 

Configs are actually under `.local/share/fcitx5/rime/` 

* default.custom.yaml
    * Actual config for customization
* installation.yaml
    * Set `sync_dir` key in this file to pick a location when using the "synchronize" feature
* user.yaml


example `default.custom.yaml` 
```
patch:
  "menu/page_size": 6
  
  ascii_composer/switch_key:
    Shift_L: commit_code # 按shift直接提交到屏幕 并且切换到英文模式
  
  switches:
    - name: simplification
      reset: 1

  schema_list:
    - schema: luna_pinyin_simp # 这样也可以选择简体

  style:
    font_point: 24           #字体大小
    color_scheme: google

    
```


Could add 
```
var:
  option:
    simplification: true
```
 to the `user.yaml` if having trouble switching to simplify mode 


### keybinding

for example if added `key_binder` seciont in `default.custom.yaml` : 

```
key_binder:
  import_preset: default
  bindings:
    - { when: composing, accept: KP_Enter, send: Return }
```

This will cause the fault set to be overwritten. Thus the soultion is to copy all the default binding out as well. 

Here is a potentional list: 
```
key_binder:
bindings:
- {accept: "Control+p", send: Up, when: composing}
- {accept: "Control+n", send: Down, when: composing}
- {accept: "Control+b", send: Left, when: composing}
- {accept: "Control+f", send: Right, when: composing}
- {accept: "Control+a", send: Home, when: composing}
- {accept: "Control+e", send: End, when: composing}
- {accept: "Control+d", send: Delete, when: composing}
- {accept: "Control+k", send: "Shift+Delete", when: composing}
- {accept: "Control+h", send: BackSpace, when: composing}
- {accept: "Control+g", send: Escape, when: composing}
- {accept: "Control+bracketleft", send: Escape, when: composing}
- {accept: "Control+y", send: Page_Up, when: composing}
- {accept: "Alt+v", send: Page_Up, when: composing}
- {accept: "Control+v", send: Page_Down, when: composing}
- {accept: ISO_Left_Tab, send: "Shift+Left", when: composing}
- {accept: "Shift+Tab", send: "Shift+Left", when: composing}
- {accept: Tab, send: "Shift+Right", when: composing}
- {accept: minus, send: Page_Up, when: has_menu}
- {accept: equal, send: Page_Down, when: has_menu}
- {accept: comma, send: Page_Up, when: paging}
- {accept: period, send: Page_Down, when: has_menu}
- {accept: "Control+Shift+1", select: .next, when: always}
- {accept: "Control+Shift+2", toggle: ascii_mode, when: always}
- {accept: "Control+Shift+3", toggle: full_shape, when: always}
- {accept: "Control+Shift+4", toggle: simplification, when: always}
- {accept: "Control+Shift+5", toggle: extended_charset, when: always}
- {accept: "Control+Shift+exclam", select: .next, when: always}
- {accept: "Control+Shift+at", toggle: ascii_mode, when: always}
- {accept: "Control+Shift+numbersign", toggle: full_shape, when: always}
- {accept: "Control+Shift+dollar", toggle: simplification, when: always}
- {accept: "Control+Shift+percent", toggle: extended_charset, when: always}
```