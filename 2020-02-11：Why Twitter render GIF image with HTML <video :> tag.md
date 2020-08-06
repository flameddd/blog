# Why Twitter render GIF image with HTML <video /> tag

## TL;DR
- size: GIFs are a terrible format for storing video.
  - reduce fils size at least 50%

## how to convert to mp4
- https://www.smashingmagazine.com/2018/11/gif-to-video/

```bash
ffmpeg -i test.gif -b:v 0 -crf 25 -pix_fmt yuv420p test.mp4
```

![img](/assets/img/gif_to_mp4.jpg)
 

## WebM
**WebM** format which offers high quality videos, often comparable to an **MP4**  

but, need to consider support (e.x. IE11 not support)
- https://caniuse.com/#search=WebM

you need to make sure that whatever **ffmpeg** build you install is compiled with **libvpx**.

```bash
ffmpeg -i animated.gif -c vp9 -b:v 0 -crf 41 video.webm
```

## HTML
This tells the browser to choose from the provided video files depending on format support. In this case, the WebM video will be downloaded and played if it’s supported, otherwise the MP4 file is used instead.

```html
<video autoplay loop muted playsinline>
  <source src="video.webm" type="video/webm">
  <source src="video.mp4" type="video/mp4">
</video>
```

To make this more robust for older browsers which do not support HTML5 video, you could add some HTML content linking to the original GIF file as a fallback.
```html
<video autoplay loop muted playsinline>
  <source src="video.webm" type="video/webm">
  <source src="video.mp4" type="video/mp4">
  <img src="animated.gif">
</video>
```

## mov to mp4
```bash
ffmpeg -i input.mov -q:v 0 output.mp4
```

## mov to gif
```bash
ffmpeg -i input.mov -pix_fmt rgb24 output.gif
```

## avi to mp4
- https://unix.stackexchange.com/questions/35746/encode-with-ffmpeg-using-avi-to-mp4

```bash
ffmpeg -i input.avi -y output.mp4
```

文章說，如果 avi 轉 mp4 時 re envode 的話，畫面、聲音可能會變差  
所以推薦用 copy mode
```bash
ffmpeg -i input.avi -c:v copy -c:a copy -y output.mp4
```