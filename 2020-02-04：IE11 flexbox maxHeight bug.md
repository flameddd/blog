# IE11 flexbox maxHeight bug

## about bug
in **IE11**, `FlexItem` would not follow (parent) `FlexBox`'s **max-height**  
```css
.Flexbox{
  max-height: 20px;
}
```
```jsx
<Flexbox>
  <FlexItem />
  <FlexItem />
  <FlexItem />
</Flexbox>
```

# solution 1
set style to child

```css
.Flexbox > FlexItem {
  max-height: 20px;
}
```

# solution 2
in `IE11`, `FlexItem` would not follow (parent) `FlexBox`'s **max-height**  

**BUT!!**, `FlexItem` CAN access the height of a `grand-parent`. **Orz|||**  

so, add Wrap it would work perfectly  

```css
.Wrap {
  display: flex;
  flex-direction: column;
}
.Flexbox {
  max-height: 20px;
}
```
```jsx
<Wrap>
  <Flexbox>
    <FlexItem />
    <FlexItem />
    <FlexItem />
  </Flexbox>
</Wrap>
```

ref: https://stackoverflow.com/questions/48709343/scrollable-flex-box-child-issue-in-ie11-edge