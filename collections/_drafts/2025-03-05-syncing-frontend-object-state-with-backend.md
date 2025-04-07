---
title: "How To Sync a Frontend JavaScript Object with a Backend"
date: 2025-03-05
layout: post
current: post
navigation: True
author: l4sh
category: development
tags:
  - javascript
  - frontend
  - backend
  - state
  - sync
  - proxy
comments: true
---

For a project I was working on I needed to partially sync a frontend JavaScript object with the backend. Meaning that I needed to send updates to the backend whenever the object was modified and update parts of the object depending on the backend response.

Usually in these types of situations you just push the data to the backend right at the point where you modify the object. But there were already a lot of moving parts interacting with this object so it would be tedious and more error prone to add the backend calls to all those places.

Doing some research I found out that you can use a [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) to monitor changes to an object and trigger a function whenever a change is made. Weirdly enough I've never had the need to use a `Proxy` before, but it was exactly what I needed.

In this article I'll outline a simple example on how to use a `Proxy` to sync an object with the backend as well as a more complex one where we fetch an initial state and go on doing partial updates back and forth.

## Frontend state object

First, we will define the state object in our frontend application. This object will hold the data that we want to sync with the backend.


```js
let state = {
  foo: 1,
  bar: 2,
  zork: "x",
  sub: {
    subfoo: 2,
    subbar: "q"
  }
};
```

```html

## Step 2 — Creating a Function to Send Updates to the Backend

Next, we will create a function that sends the updated state to the backend server whenever the state object is modified.

```js
function updateBackend(state) {
  fetch('/api/update-state', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(state)
  })
  .then(response => response.json())
  .then(data => console.log('Success:', data))
  .catch((error) => console.error('Error:', error));
}
```

## Step 3 — Monitoring Changes to the State Object

To automatically send updates to the backend whenever the state object is modified, we can use a Proxy to monitor changes to the object.

```js
let handler = {
  set: function(target, property, value) {
    target[property] = value;
    updateBackend(state);
    return true;
  }
};

let proxiedState = new Proxy(state, handler);
```

## Step 4 — Updating the State Object

Now, whenever we update the `proxiedState` object, the changes will be sent to the backend server.

```js
proxiedState.foo = 3; // This will trigger the updateBackend function
proxiedState.sub.subfoo = 5; // This will also trigger the updateBackend function
```

## Conclusion

In this article, you learned how to sync a frontend JavaScript object with a backend server using a Proxy. This ensures that any changes made to the state object on the frontend are automatically sent to the backend, keeping your application state consistent.

You can further explore this technique by adding more complex state management and error handling to suit your application's needs.
