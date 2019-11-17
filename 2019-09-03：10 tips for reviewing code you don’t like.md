## 10 tips for reviewing code you don’t like
### David Lloyd July 8, 2019
#### https://developers.redhat.com/blog/2019/07/08/10-tips-for-reviewing-code-you-dont-like

A code review
- should be objective
- and concise and should deal in certainties whenever possible
- It’s not a political or emotional argument

the goal should always be to move forward and elevate the project and its participants

### 1. Rephrase your objection as a question

Bad:
- “This change will make XXX impossible.”
  - (This is hyperbole; is it really impossible?)

Good:
- “How can we do XXX with your change?”

### 2. Avoid hyperbole
- Simply state your concerns
- and ask questions to help get to the desired outcome.

Bad:
- “This change will destroy performance.”

Good:
- “It seems like doing X might be slower than existing Y; have you measured/gathered data to show it isn’t?”

Better (if you have time):
- “In the meantime, I am gathering data to try to verify that X is not slower than Y.”

Also good:
- “This change changes this single loop O(n) to a doubly nested loop O(n²); won’t this affect performance?”

### 3. Keep snide comments to yourself
Some thoughts are better kept to yourself.

Bad:
- “I think this change is bad and will ruin everything.”
- “Are you sure that software engineering is the right career path for you?”

### 4. Engage positively
If you engage positively, you might end up discovering a solution that is better than either original option.

Bad:
- “This change sucks, my version is better.”

Good:
- “I also have a similar change at this location XXX: maybe we can compare and/or combine ideas.”

Also good:
- “I have a similar change in progress, but I chose to do X because ZZZ; why did you choose Y?”

### 5. Remember that not everybody’s experience is identical to yours
不是所有的人 knowledge base 都一樣的！

Bad:
- “Can’t you see that this is obviously wrong?”

Good:
- “This is incorrect because it causes a null pointer exception when X is Y.”

### 6. Don’t diminish the complexity of something that’s not obvious
提出不同的解法

Bad:
- “Why not simply frob the gnozzle?”

Good:
- “It might be possible to frob the gnozzle, which would simplify this part (see XXX for an example).”

### 7. Be respectful
PR 不符合最基本要求的品質的話，就讓它知道吧  
但，保持尊重

Bad:
- “This is stupid code written by a stupid person.”

Good:
- “Thanks for your contribution. However, it cannot be accepted in its current form; there are multiple problems (as outlined above).”

Also good:
- “As outlined above, there are multiple problems with this submission.  Maybe we could back up a step and talk about the use cases instead?  That could help us find a path forward.”

### 8. Manage expectations (and your time)
有人提太大包的話，就讓他知道吧。請他拆開

Bad:
- “I’m not merging this, it’s too big.”
- Ignoring it until it goes away.

Good:
- “Could you please break this down into smaller changes? I do not have a lot of time for code reviews and this one is just too large/complex to review in one pass.”

### 9. Say please
說個「請」，表示對 sumitter 的付出的尊重

- “Could you please separate the whitespace changes into another pull request?”
- “Could you please align these variable definitions so they’re easier to read?”

### 10. Start a conversation
如果你不喜歡該 PR，但你沒有很清楚你討厭的原因  
那不訪 Start a conversation 吧  

it’s also okay to say
- “I don’t like this and I’m not sure why, can we talk about it?”

Even skilled and experienced engineers should be able to say
- “I don’t understand why I don’t like this”

it’s not an invitation to attack the position of the reviewer but rather an honest quest for knowledge.