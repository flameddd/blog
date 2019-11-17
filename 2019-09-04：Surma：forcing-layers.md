## Surma：forcing-layers
### Surma
#### https://dassur.ma/things/forcing-layers/

browser 的 rendering engine 簡化流程為
1. Download and parse HTML, generating a DOM tree.
2. Process styling to lay out the document, generating a “layout tree”.
3. Turn the layout tree into paint instructions, generating a “paint tree”.
4. Generate a canvas big enough to hold the entire document.
5. Execture all those paint instruction on that canvas.

當我們要 rotating 某個 element 時
- we can’t really reuse the old canvas.
- have to start fresh and go all the way back to step 2.

所以有些優化方式會讓 相關的 render 到另一 layers 去


```css
{
  transform: translateZ(0)
}
```

`translateZ`, as it will use the GPU to calculate the perspective distortion (even if it ends up being no distortion at all).

但這方法會 make elements flicker in Chrome and Safari (that is not the case anymore)

所以其他會有建議用

```css
{
  backface-visibility: hidden;
}
```

後來 css 支援 ` will-change` 屬性，讓 browser 可以提前優化  
（`Edge` 不支持。新版的 Edge 因為是 Chrome 核心了，應該可以）


### 結論

最優先用
```css
{
  will-change: 'opacity';
}
```

或者是

```css
{
  backface-visibility: 'hidden';
}
```

這兩種方法來 to force an element onto its own layer as it seems like the side-effects are the most unlikely to be a problem

而當你有用到 `transform` 時  
就改用  
```css
{
  will-change: 'transform';
}
```
