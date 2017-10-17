# React Setup in Rails

In the past, Launchpad Lab has used the
[react_on_rails](https://github.com/shakacode/react_on_rails) gem to great effect.

Prior to Rails 5.1, the `react_on_rails` gem provided an elegant solution for
integrating React in a Rails application.

## Webpacker

As of Rails 5.1, we have a new flag option for automagically installing the
Webpack and React in our new Rails application:

```sh
$ rails new app-name --webpack=react
```

If you'd like to install React within an existing Rails 5.1 application:

```
$ rails webpacker:install:react
```

This includes and bundles [Webpacker](https://github.com/rails/webpacker) in
our gemfile, which then runs the Webpack setup and React installation. 
In addition, the generator provides a "Hello World" React component.

```
app
.
|____javascript <---- React installation
| |____packs
| | |____application.js
| | |____hello_react.jsx
|____views
| |____layouts
| | |____application.html.erb
| | |____mailer.html.erb
| | |____mailer.text.erb
| |____pages
| | |_____home.html.erb
```

In *javascript/packs/hello_react.jsx*

```javascript

	// Run this example by adding <%= javascript_pack_tag 'hello_react' %> to the head of your layout file, like app/views/layouts/application.html.erb.

	import React from 'react'
	import ReactDOM from 'react-dom'
	import PropTypes from 'prop-types'

	const Hello = props => (
	  <div>Hello {props.name}!</div>
	)

	Hello.defaultProps = {
	  name: 'David'
	}

	Hello.propTypes = {
	  name: PropTypes.string
	}

	document.addEventListener('DOMContentLoaded', () => {
	  ReactDOM.render(
	    <Hello name="React" />,
	    document.body.appendChild(document.createElement('div')),
	  )
	})
```

Webpacker by default provides a helper method `javascript_pack_tag` to help us
render our components. The method takes a string as an argument, which points to
the name of the React component you'd like to render.

```erb
	<%=  javascript_pack_tag 'hello_react' %>
```

This approach is similar to the method provided by react_on_rails: `react_component()`, but does not provide a way to pass props.

That's a bit of a problem.

## Webpacker-React

Fortunately there is a follow up gem, called
[Webpacker-React](https://github.com/renchap/webpacker-react) which bridges the
gap between your Rails backend and Webpacker.

Webpacker-React provides the helper method `react_component()`, which can be
used in either the view or the controller to render your React components - and
we can pass in props!

*Rendering From A View*

```erb
<%= react_component('Hello', name: 'React') %>
```
*with html element*
```erb
<%= react_component('Hello', { name: 'React' }, tag: :span, class: 'my-custom-component') %>
```
*will render*
```html
<span class="my-custom-component" data-react-class="Hello" data-react-props="..."></span>
```

*Rendering From A Controller*

```ruby
class PageController < ApplicationController
  def main
    render react_component: 'Hello', props: { name: 'React' }
  end
end
```
*with html element*
``` ruby
	render react_component: 'Hello', props: { name: 'React' }, tag_options: { tag: :span, class: 'my-custom-component' }
```
*will render*
```html
<span class="my-custom-component" data-react-class="Hello" data-react-props="..."></span>
```

Within our `javascript` directory, there is a bit more work to be done to ensure
`webpacker-react` works as expected.

Our folder structure:

```
app
.
|____javascript <--- React installation
| |____components
| | |____hello.jsx
| |____packs
| | |____application.js
| | |____hello_react.js
|____views
| |____layouts
| | |____application.html.erb
| | |____mailer.html.erb
| | |____mailer.text.erb
| |____pages
| | |_____home.html.erb

```

In *javascript/components/hello.jsx*

```javascript
import React from 'react';

const Hello = (props) => {
  return (
    <div>
      Hello, I am {this.props.name}!
    </div>
  )
}

export default Hello;
```

In *javascript/packs/hello_react.js*

```javascript
	import Hello from 'components/hello';
	import WebpackerReact from 'webpacker-react';

	WebpackerReact.setup({Hello});
```

We need to use the `WebpackerReact.setup()` function to register all desired components within our `hello_react.js` pack.

To make `hello_react.js` available to the view, we still need our Webpacker `javascript_pack_tag` method in the `application.html.erb` head:

```erb
	<!DOCTYPE html>
	<html>
	  <head>
	    <title>RailsReact</title>
	    <%= csrf_meta_tags %>

	    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
	    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
	    <%= javascript_pack_tag 'hello_react' %>
	  </head>

	  <body>
	    <%= yield %>
	  </body>
	</html>
```

Lastly we render our Hello component with `react_component` method in the view:

In *views/pages/home.html.erb*

```erb
	<%= react_component('Hello', name: 'a component rendered from a view') %>
```

## Disclaimer 

This is most basic React setup possible in Rails 5.1, and does not include the setup requirements of `lp-components`, `redux-form`, or `lp-redux-api`.

Please see the "react.md" section of this guide to walk through how we organize, configure, and ultimately build React components.

Check out our [Todo MVC](https://github.com/LaunchPadLab/lp-todo-mvc) example which features a more robust React/Webpack setup and Rails API.
