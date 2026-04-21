
## adjust metadata

```
file="Marc-Uwe Kling - 01 - Kapitel 43 - Orientierungslos.mp3"
gesamtzahl=19
albumtitel="Qualityland"
interpret="Marc-Uwe Kling"
~/.local/bin/eyeD3 $file -a $interpret -A $albumtitel -N $gesamtzahl --add-image "qualityland.png:FRONT_COVER" -t "$(echo $file | sed "s/\.mp3//g" | sed "s/^......................//g")" -n "$(echo $file | sed "s/\.mp3//g" | cut -d " " -f 4)"
```
