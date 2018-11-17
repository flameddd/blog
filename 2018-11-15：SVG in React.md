
# Flexible Icons with React and SVG
來源：[https://open.nytimes.com/flexible-icons-with-react-svg-973f310e6382](https://open.nytimes.com/flexible-icons-with-react-svg-973f310e6382)

`Scott Taylor`(就職於 `New York Times`) 分享 SVG in React 的範例。

## Manipulating and optimizing SVG
透過一些工具來最佳化 SVG
 - [svgomg](https://jakearchibald.github.io/svgomg/)

## example
這種做法跟我們目前的差不多，這邊多寫這邊多點記憶。

```javascript
import React from 'react';
import PropTypes from 'prop-types';

function TLogo({ className, fill }) {
  return (
    <svg viewBox="0 0 16 16" className={className}>
      <path fill={fill} d="M13.754 9.776c-.485 1.281-1.378 2.27-2.658 2.794V9.776l1.533-1.378-1.534-1.358V5.118c1.397-.097 2.367-1.126 2.367-2.387 0-1.649-1.572-2.232-2.464-2.232-.194 0-.408 0-.718.078v.078c.116 0 .291-.019.349-.019.621 0 1.087.291 1.087.854 0 .427-.349.854-.97.854-1.533 0-3.338-1.242-5.298-1.242-1.746 0-2.95 1.3-2.95 2.62 0 1.3.757 1.727 1.552 2.018l.02-.078c-.252-.156-.426-.427-.426-.854 0-.582.543-1.067 1.223-1.067 1.649 0 4.308 1.378 5.957 1.378h.155V7.06L9.446 8.398l1.533 1.378v2.833c-.64.233-1.3.33-1.979.33-2.561 0-4.191-1.553-4.191-4.133 0-.621.078-1.223.252-1.805l1.281-.563v5.705l2.6-1.145V5.157L5.118 6.865c.388-1.125 1.184-1.94 2.135-2.406L7.234 4.4c-2.562.563-5.046 2.504-5.046 5.414 0 3.357 2.833 5.686 6.132 5.686 3.493 0 5.472-2.329 5.492-5.724h-.058z"/>
    </svg>  
  );
}

TLogo.propTypes = {
  className: PropTypes.string,
  fill: PropTypes.string,
};

TLogo.defaultProps = {
  className: undefined,
  fill: '#333',
};

export default TLogo;
```

使用時就這樣
```javascript
import React from 'react';
import { css } from 'emotion'; 
import TLogo from './TLogo';

const black = '#000';
const logoClass = css`
  height: 24px;
  width: 24px;

  @media (min-width: 800px) {
    height: 48px;
    width: 48px;
  }
`;

export default function Header() {
  return (
    <header>
      <TLogo fill={black} className={logoClass} />
    </header>
  );
}
```

還可以這樣改 `path`
```javascript
import { css } from 'emotion';

const black = '#000';
const logoClass = css`
  height: 24px;
  width: 24px;
  path {
    fill: ${black};
  }
  @media (min-width: 800px) {
    height: 48px;
    width: 48px;
  }
`;
```