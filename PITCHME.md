# Redux Best Practices
---
## Constants
+++
### What are constants for?
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
### Problem?
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
### Possible solution:
### don't refer to constants, only to actions
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
+++
@quote[Many reducers may handle one action. One reducer may handle many actions. Putting them together negates many benefits of how Flux and Redux application scale. This leads to code bloat and unnecessary coupling. You lose the flexibility of reacting to the same action from different places, and your action creators start to act like “setters”, coupled to a specific state shape, thus coupling the components to it as well.](Dan Abramov)
+++
@quote[The point of having reducers is that actions aren't coupled to reducers.](Dan Abramov)
+++
@quote[The point of Flux and Redux is to make it easy to suddenly start reacting to the same action in a different place. You don’t know which actions you’ll need to handle in different places in advance.](Dan Abramov)
+++
@quote[This patterns grinds your productivity to a screeching halt when requirements change and your application suddenly needs to dispatch actions and respond to slices of your state that break this pattern. That’s what makes this pattern so dangerous. It’s important to recognize the red-flags associated with this pattern before it turns into a daunting task to rework your application state.](Steven Scaffidi)

---
## connect() - how often?
+++
### Should I only connect my top component, or can I connect multiple components in my tree?
+++
@quote[Emphasizing “one container component at the top” in Redux examples was a mistake.]
@quote[Whenever you feel like you're duplicating code in parent components to provide data for same kinds of children, time to extract a container. Generally as soon as you feel a parent knows too much about “personal” data or actions of its children, time to extract a container.](redux.js.org FAQ)
---
## Component/container separation
+++
@quote[You generally have "app-specific components that need to be connected to Redux", and "truly generic components that probably aren't going to get connected, or if they are, will be connected many times for different parts of the app" so if you're trying to structure things as, say, `components/UserList/UserList.jsx`, and `containers/UserListContainer/UserListContainer.jsx`, that's overkill. Just put both of them in one file, `export class UserList`, and `export default connect(mapState)(UserList)`](Mark Erikson)
---
## Immutability
+++
### Best Practices for using Immutable.JS with Redux?
@ul
- Never mix plain JavaScript objects with Immutable.JS
- Limit your use of toJS()
- Never use toJS() in mapStateToProps
- Use Immutable.JS objects in your Smart Components
- Never use Immutable.JS in your Dumb Components
- Use a Higher Order Component to convert your Smart Component’s Immutable.JS props to your Dumb Component’s JavaScript props
@ulend
+++
### Mutation?
If we never change the store directly, only through "CRUD"-style actions/reducers, how would we ever mutate the store?
###
toJS() bad in mapStateToProps()?
+++
### Alternative: redux-immutable-state-invariant

```javascript
import immutableStateInvariant from 'redux-immutable-state-invariant';
const enhancer = applyMiddleware(
  someOtherMiddleware,
  immutableStateInvariant()
);
const store = createStore(reducer, enhancer);
```
+++
    A state mutation was detected between dispatches,
      in the path: ui.loader

    A state mutation was detected inside a dispatch,
      in the path: data.entities
---
## (Custom) Middleware
+++
```javascript
const noopMiddleware = store => next => action => {
  return next(action)
}
```
+++
```javascript
// logger middleware
// console.debug the whole action
const logger = store => next => action => {
  console.debug("%o",action);
  return next(action);
};
```
+++
```javascript
import { addFlashMessage, removeFlashMessage } from '../actions/';
/* flash messages middleware
  checks for meta.flash
  dispatches addFlashMessage
  and schedules the removal of the flash message
  after meta.flash.duration seconds */
let nextFlashMessageId = 1;
const flash = ({ getState, dispatch }) => next => action => {
  if (action.meta && action.meta.flash) {
    const { type, text, duration } = action.meta.flash;
    const id = nextFlashMessageId;
    nextFlashMessageId += 1;
    dispatch(addFlashMessage(id, type, text, duration));
    if (duration !== false) {
      setTimeout(() => {
        dispatch(removeFlashMessage(id));
      }, duration);
    }
  }
  return next(action);
};
```
---
## Selectors
+++
### access to piece of state
```javascript
const isInternational = campaign.getIn(['country', 'isAddressValidationSupported']);

const showEmptyListInfoContent =
  showEmptyListInfo
  && automation.get('campaigns').size === 0
  && unvalidated === 0
  && deliverable === 0
  && undeliverable === 0;
```
+++
### selector functions
```javascript
// Reducer
const todo = (state = [], action) => {
 switch (action.type) {
   case GET_ALL_TODOS:
     return {
       todos: action.data.todos
     };
   default:
     return state;
 }
}
// Selector
const getIncompleteTodos = state => 
  state.todos.filter(todo => !todo.completed);
```
+++
### potential performance problems? reselect
```javascript
import { createSelector } from 'reselect'

const shopItemsSelector = state => state.shop.items
const taxPercentSelector = state => state.shop.taxPercent

const subtotalSelector = createSelector(
  shopItemsSelector,
  items => items.reduce((acc, item) => acc + item.value, 0)
)

const taxSelector = createSelector(
  subtotalSelector,
  taxPercentSelector,
  (subtotal, taxPercent) => subtotal * (taxPercent / 100)
)

export const totalSelector = createSelector(
  subtotalSelector,
  taxSelector,
  (subtotal, tax) => ({ total: subtotal + tax })
)
```
---
## Data normalization
+++
### normalizr
+++
```javascript
{
  "id": "123",
  "author": {
    "id": "1",
    "name": "Paul"
  },
  "title": "My awesome blog post",
  "comments": [
    {
      "id": "324",
      "commenter": {
        "id": "2",
        "name": "Nicole"
      }
    }
  ]
}
```
+++
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
+++
```javascript
{
  result: "123",
  entities: {
    "articles": {
      "123": {
        id: "123",
        author: "1",
        title: "My awesome blog post",
        comments: [ "324" ]
      }
    },
    "users": {
      "1": { "id": "1", "name": "Paul" },
      "2": { "id": "2", "name": "Nicole" }
    },
    "comments": {
      "324": { id: "324", "commenter": "2" }
    }
  }
}
```
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
