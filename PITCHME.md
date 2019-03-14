# Redux Best Practices
---
# Constants
+++
## Why are we using constants?
```javascript
// constants/todo.js
const ADD_TODO = 'ADD_TODO'

// actions/todo.js
import { ADD_TODO } from '../constants/todo'
const addTodo = title => (
  {
    type: ADD_TODO,
    payload: title
  }
)
export addTodo
```
@[1-2](constant)
@[3-10](action)
+++
## Problem?
```javascript
// constants/view/campaign/list.js
import defineActionTypes from '../../../../utils/defineActionTypes';

export default defineActionTypes('CAMPAIGN_LIST', {
  DELETE: `
    SUBMIT
  `,
});

// sagas/view/campaign/list.js
import T from '../../../constants/view/campaign/list';

function* submitDeleteSaga() {
  yield takeLatest(T.DELETE.SUBMIT, submitDelete);
}
```
@[1-8](object of constants)
@[10-15](saga)
@[14](runtime error? silent error?)
+++
## Solution
```javascript
import { createAction } from 'redux-starter-kit'
const actionCreator = createAction("SOME_ACTION_TYPE");

console.log(actionCreator.toString())
// -> "SOME_ACTION_TYPE"

console.log(actionCreator.type);
// -> "SOME_ACTION_TYPE"
```
---
## Reducers and action creators aren't a one-to-one mapping
@quote[Many reducers may handle one action. One reducer may handle many actions. Putting them together negates many benefits of how Flux and Redux application scale. This leads to code bloat and unnecessary coupling. You lose the flexibility of reacting to the same action from different places, and your action creators start to act like “setters”, coupled to a specific state shape, thus coupling the components to it as well.](Dan Abramov)
---
# Middleware
---
# Selectors
+++
### reselect
---
@transition[convex concave]
# Data normalization
+++
@transition[slide-in zoom-out]
### normalizr
```javascript
import { normalize, schema } from 'normalizr';

// Define a users schema
const user = new schema.Entity('users');

// Define your comments schema
const comment = new schema.Entity('comments', {
  commenter: user
});

// Define your article
const article = new schema.Entity('articles', {
  author: user,
  comments: [comment]
});

const normalizedData = normalize(originalData, article);
```
@[1](Import)
@[3-8](Define schemas)
@[17](Run it)
---
# FA icons
@fa[thumbs-up](Sounds good to me!)
@fab[docker]
@fab[digital-ocean text-bold](Digital Ocean)
@fa[umbrella fa-7x fa-flip-horizontal fa-pulse]
---
# Lists
+++
## Unordered
@ul

- Plain text list item
- Rich **markdown** list *item*
- Link [within](https://gitpitch.com) list item

@ulend
+++
## Ordered
@ol

- Plain text list item
- Rich **markdown** list *item*
- Link [within](https://gitpitch.com) list item

@olend
---
Images
![Logo](assets/img/yorick.jpg)
![Logo](https://www.dvdplanetstore.pk/wp-content/uploads/2018/01/bYkwHCFjJ18EYSOMS1NwlhoVa8C.jpg)
---
## Add Some Slide Candy

![](assets/img/presentation.png)

---
@title[Customize Slide Layout]

@snap[west span-50]
## Customize Slide Content Layout
@snapend

@snap[east span-50]
![](assets/img/presentation.png)
@snapend

---?color=#E58537
@title[Add A Little Imagination]

@snap[north-west]
#### Add a splash of @color[cyan](**color**) and you are ready to start presenting...
@snapend

@snap[west span-55]
@ul[spaced text-white]
- You will be amazed
- What you can achieve
- *With a little imagination...*
- And **GitPitch Markdown**
@ulend
@snapend

@snap[east span-45]
@img[shadow](assets/img/conference.png)
@snapend

---?image=assets/img/presenter.jpg

@snap[north span-100 headline]
## Now It's Your Turn
@snapend

@snap[south span-100 text-06]
[Click here to jump straight into the interactive feature guides in the GitPitch Docs @fa[external-link]](https://gitpitch.com/docs/getting-started/tutorial/)
@snapend
