
## adjust metadata

```
file="Marc-Uwe Kling - 01 - Kapitel 43 - Orientierungslos.mp3"
albumtitel="Qualityland"
interpret="Marc-Uwe Kling"
cover="qualityland.jpeg"
genre="Hörbuch"
releaseYear=2017

titel=$(echo $file | sed "s/\.mp3//g" | sed "s/^......................//g")
tracknumber=$(echo $file | sed "s/\.mp3//g" | cut -d " " -f 4)

gesamtzahl=12
discnumber=1

~/.local/bin/eyeD3 $file -a $interpret -A $albumtitel -N $gesamtzahl --add-image "$cover:FRONT_COVER" -t "$titel" -n $tracknumber -d $discnumber --genre "$genre" --release-year $releaseYear --preserve-file-times
```

## get cover

```
read "coverURL?Cover file URL: "                                                                                                                                                         read "cover?Cover file name: "                                                                                                                                                             wget -O $cover $coverURL
```

## adjust whole folder

```
for file in *.mp3
do
  ~/.local/bin/eyeD3 --remove-all $file
  titel=$(echo $file | sed "s/\.mp3//g" | sed "s/^......................//g")
  tracknumber=$(echo $file | sed "s/\.mp3//g" | cut -d " " -f 4)
  ~/.local/bin/eyeD3 $file -a $interpret -A $albumtitel -N $gesamtzahl --add-image "$cover:FRONT_COVER" -t "$titel" -n $tracknumber -d $discnumber --genre "$genre" --release-year $releaseYear --preserve-file-times
done
```

## read metadata

```
ffmpeg -i "Marc-Uwe Kling - 01 - Versionsangabe.mp3" -f ffmetadata metadata.txt
```

## remove metadata

```
~/.local/bin/eyeD3 --remove-all $file
```

