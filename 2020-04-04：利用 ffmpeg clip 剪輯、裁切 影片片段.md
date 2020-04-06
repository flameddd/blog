# 利用 ffmpeg clip 剪輯、裁切 影片片段

當
- 想要 clip 影片片段
- 不改變 encode

時，可用這指令

```bash
ffmpeg -ss 00:01:00 -i input.mp4 -to 00:02:00 -c copy output.mp4
```

`-c` 方式是 copy 影片的方式，這樣速度超快  
