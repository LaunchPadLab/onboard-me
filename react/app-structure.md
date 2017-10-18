# React-Rails LPL App Structure

At LPL we strive to follow the same pattern in our react apps, just the way we do with traditional Rails apps.

You will likely see patterns similar to this one in other app examples, with a few nuances for the *LPL way*.

The basic folder structure is as follows:

```
app
.
.
.
|____javascript
| |____components
| |____main
| | |____todo
| | | |____components
| | | | |______index.js
| | | | |______todo-list-item.js
| | | | |______todo-list.js
| | |______forms
| | | | |______index.js
| | | | |______item-form.js
| | |______views
| | | | |______index.js
| | | | |______todo.js
| | |______actions.js
| | |______index.js
| | |______reducer.js
| | |______routes.js
| | |____api-actions.js
| | |____api.js
| | |____auth.js
| | |____effects.js
| | |____index.js
| | |____layout.js
| | |____reducer.js
| | |____routes.js
| | |____store.js
| | |____types.js
|____utils
| |____index.js
.
.
.
|____views
| |____pages
| | |_____home.html.erb
```


<!-- a few high level details of some of the files, and point out how we funnel the order of operations through the views folder which does all of the composing, etc. -->
