# relative timestamp description (react, redux example)
### https://redux.js.org/tutorials/essentials/part-4-using-data

This is redux documentation's example to how to show **relative timestamp description**.  
I think this example is very simple and clean. It's worth to take note.  
Redux documentation has full example.  

relative timestamp description, such like
- `5 minutes ago`
- `10 minutes ago`
- `now`
- `6m`
- `1h`
- `16h`

## step1
add **time stamp** when add post
```js
function newPost(title, content, userId) {
  return {
    payload: {
      id: nanoid(),
      date: new Date().toISOString(), // <-- add newPost's time stamp
      title,
      content,
      user: userId,
    },
  }
}
```

## step2
parse **time stamp** as **relative timestamp description**
- this example is relay on `date-fns`
- if `date-fns` not fit your require, write your formatter or use other liber
  - compare the `date` or use `localeCompare`
  - `timeStamp.localeCompare(new Date().toISOString())`
 
```js
import React from 'react'
import { parseISO, formatDistanceToNow } from 'date-fns'

export const TimeAgo = ({ timestamp }) => {
  let timeAgo = ''
  if (timestamp) {
    const date = parseISO(timestamp)
    const timePeriod = formatDistanceToNow(date) // parse to "relative timestamp"
    timeAgo = `${timePeriod} ago`
  }

  return (
    <span title={timestamp}>
      &nbsp; <i>{timeAgo}</i>
    </span>
  )
}
```


## re-order render if in need

```js
function Posts ({posts , ...props}) {
  const orderedPosts = posts.slice().sort((a, b) => b.date.localeCompare(a.date))

  const renderedPosts = orderedPosts.map(post => {
    return (
      <article className="post-excerpt" key={post.id}>
        <h3>{post.title}</h3>
        <div>
          <PostAuthor userId={post.user} />
          <TimeAgo timestamp={post.date} />
        </div>
        <p className="post-content">{post.content.substring(0, 100)}</p>
        <Link to={`/posts/${post.id}`} className="button muted-button">
          View Post
        </Link>
      </article>
    )
  })
}
```
