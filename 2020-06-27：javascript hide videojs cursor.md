# use javascript hide `videojs` cursor when not fullscreen

`videojs` dose not hide `cursor` when playing (not fullscreen)  
this is a demo to how to use `MutationObserver` to observer `videojs` and detect when to hide cursor
- I implement this feature in `"video.js": "^7.6.6"`
- the best timing to hide `cursor` is follow `control bar`
  - hide `cursor` when `control bar` hide
  - show `cursor` when `control bar` show
  - `control bar` hide only when status are **"vjs-playing"** and **"vjs-user-inactive"**

(demo code is react version)
```html
<div data-vjs-player>
  <video className="video-js"></video>
</div>
```

```js
const observer = new MutationObserver(mutations => {
  // only need chech latest mutation's classname
  const mutatedClassName = mutations[mutations.length - 1].target.className;
  if (
    mutatedClassName.indexOf("vjs-playing") !== -1 &&
    mutatedClassName.indexOf("vjs-user-inactive") !== -1
  ) {
    // hide cursor when video playing and user-inactive
    // in my case, I hide the parent element's cursor.
    // you can just hide the videojs element.
    document.querySelector('#id').style.cursor = "none";
  } else {
    // I knew that "prv-line's else" and "next-line's if" can combin together
    // I separate the code here for better logic readability
    if (document.querySelector('#id').style.cursor !== "none") {
      // videojs element change className very often
      // so, 「if !== "none" 」 is for avoid unnecessary value set
      document.querySelector('#id').style.cursor = "auto";
    }
  }
});

// "#vjs_video_3" is videojs's element ID
// make sure you observe right element
observer.observe(document.querySelector("#vjs_video_3"), {
  attributes: true,
  attributeFilter: ["class"]
});

// remember disconnect when observer no need anymore
// like event listener
observer.disconnect()
```

## demo
- OtterPlayer: https://flameddd.github.io/OtterPlayer/
  - play a local video and observer cursor behavior 
