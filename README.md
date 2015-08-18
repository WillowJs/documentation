# Why build Willow?

Over the past four years I've completely transitioned to node as my web application backend language of choice. Node is fast and allows me to use the same language for both the frontend and the backend. However, there aren't many tools that truely take advantage of the full power that comes with being able to use JavaScript for everything. I'm building Willow because I was looking for a lightweight tool that allowed me to do the following:

* Reuse code on the frontend an backend. Specifically I was interested in reusing validation logic and views. But Willow is not limited to sharing these two categories.
* Easily reuse code from one application in other application. For example, I shouldn't have to rebuild a login page, registration page, and recover password flow for each new application.
* Take advantage of server side rendering for page load speed and SEO. Willow uses React to automatically attaches event handlers on the client side without completely re-rendering the page.
* Be able to use all of the great packages that are published on npm.

# How does Willow work?

Willow leveradges the great work that's already been done in the JavaScript community and pulls it together in an integrated set of modules. The structure of Willow applications is modeled after (and heavily uses) React. Willow introduces a new type of event handlers to React components. Each handler specified a name, a method (get, put, post, delete or local), a list of dependencies (other handlers that must be run before itself) and a run method. Two Willow event handlers 

 that are identified by the developer to be run either on the server-side or client-side. Each event handler also lists dependencies. Dependencies are other handlers that must be run first. Defining a two willow event handlers that respond to a `foo` event would look like this:

```js
Willow.createClass({ /* React code here */ })
.on('foo', {
	name: 'bar',
	method: 'local',
	dependencies: [],
	run: function(e, resolve, reject) {
		// handler code goes here...
	}
})
.on('foo', {
	name: 'baz',
	method: 'post',
	dependencies: ['bar'],
	run: function(e, resolve, reject) {
		// handler code goes here...
	}
});
```

The properties on these event handlers are...

* `name` uniquely identifies the handler
* `method` determines where and how the run method will execute. It can be either local (runs on the client), post (runs on the server via AJAX post request), get, put or delete.
* `dependencies` is an array of handler names. A handlers run method will only execute once all of the handlers listed as dependencies have resolved.
* `run` method that contains the code to execute. The run method takes an event object, resolve and reject callbacks as its arguments. Any data that is passedinto the resolve callback will be merged into the event object for future handlers to use.

Submitting a simple form. The flow might look like:

1. HTTP request is made to the server. Node renders the HTML for the component (form) and sends it to the client.
2. React on the client attaches the appropriate event listeners to the server rendered HTML.
2. User fills in the form.
3. User clicks the submit button and a submit event is fired.
4. A client-side `validate` handler responds to that submit event and validates that the data in the form is good.
5. A server-side `save` handler is also set up to respond to the submit event, but has a dependency of the `validate` handler. It runs after the `validate` handler is successful, revalidating before saving the form data to the server.

Both the `validate` handler and the `save` handler are defined on the Willow component, but one runs on the client, and the other runs on the server. One of the great things about this structure is that the form component can be dropped into any part of the application (or an entirely new application) and should work seamlessly because it is entirely self contained.


Willow uses...

* express middleware for server side routing and processing
* react for client-side and server-side rendering
* gulp for pre-processing of components
* async for running event handlers in the correct order
* browserify for module loading

# In a nutshell

Willow is a set of JavaScript modules that allow you to build self contained isomorphic components.

# Examples

For examples of different components that have been built with Willow check out the [examples repository](https://github.com/WillowJs/examples).

# Outstanding questions

- How do we store data consistently among components that might use different databases? We don't want to use 4 different external components that all use a different data store. Ideally each component would integrate with the data store that we are using for the application. Alternatively we could force postgres on everyone, force use of knex, or handle all storage at the application level through an event system (flux-like architecture).
- If two different components both use the same npm module on the client how do we only include that module once as opposed to once for each component that is using it?

# Solved questions

- How to prevent server side code from being shared on the client?
	- Gulp auto-generates client side versions of each component that have server side logic stripped out.