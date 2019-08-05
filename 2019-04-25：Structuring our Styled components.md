## Structuring our Styled components
### Alan B Smith Dec 9, 2017
#### https://tech.decisiv.com/structuring-our-styled-components-part-i-2bf21fa64b28
```
├ src/
├── blocks/
| ├── Card/
| | ├── Header.js
| | ├── Image.js
| | ├── Text.js
| | ├── Title.js
| | └── index.js // <- Block
```

```js
// src/blocks/Card/index.js

import styled from 'styled-components';

import Header from './Header';
import Image from './Image';
import Text from './Text';
import Title from './Title';

const Card = styled.div`
  background: #ffffff;
  border-radius: 2px;
  margin: 5px 5px 10px;
  padding: 5px;
  position: relative;
  box-shadow: 2px 2px 4px 0px #cfd8dc;
`;

Card.Header = Header;
Card.Image = Image;
Card.Text = Text;
Card.Title = Title;

export default Card;
```

```js
// SomeComponent.js

import Card from './blocks/Card';

const Container = () => (
  <Card>
    <Card.Header>
      <Card.Image
        alt=”bob-ross-headshot”
        src=”www.example.com/bob-ross.jpg”
      />
      <Card.Title>
        Bob Ross
      </Card.Title>
    </Card.Header>
    <Card.Text>
      Robert Norman Ross (October 29, 1942 – July 4, 1995) was an American painter,
      art instructor, and television host. He was the creator and host of
      The Joy of Painting, an instructional television program that aired from
      1983 to 1994 on PBS in the United States…
    </Card.Text>
  </Card>
)
```