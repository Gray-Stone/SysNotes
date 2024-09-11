

```
sudo apt install fctix5 fctix5-rime
```

A reboot might be needed after getting these setup

## KDE integrated settings menu 

kde-config-fcitx5


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

