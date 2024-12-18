# whisper.cpp

## Windows

Binaries can be downloaded from...

With GPU if possible much better...

Run

```bash
./main -l de -m ggml-large-v3-turbo.bin -f my.wav > test.txt
```

Run in chunks...

1. `ffmpeg -i _DevDay\ -\ TA-20241113_144649-Besprechungsaufzeichnung.mp4 -vn -acodec pcm_s16le -ar 16000 output-dimi.wav` in wav verwandeln
1. Dauer wav checken `ffmpeg -i output-olli.wav 2>&1 | grep "Duration"`
1. Shell scripts chunks anpassen
