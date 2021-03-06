[Home](https://github.com/icarus-sullivan/redux-config/blob/master/README.md)

# createActions
Configuration for actions can be done in multiple ways to make edge cases easier to handle.

Options:

| key| value | required | default |
|--|--|--|--|
| type | string | yes | - |
| invocationType | string | - | 'sync' |
| fn | function | async only | - |
| errorTransform | AsyncFunction | - | - |
| payload | any | static only | - |

** Note actions can also be normal functions, these will be wrapped in dispatch and invoked and should follow the normal `{ type, payload }` convention when applicable.

### Async Declarations - Success
There is a lot of boilerplate surrounding requesting, receiving, failure and final states in async requests. Based off conventions for web request, dispatched actions are called automatically, and are derived from the initial type. 

```javascript
import { createActions } from '@sullivan/redux-config';

const dispatch = console.log;

const fetch = (arg) => new Promise((resolve) => {
  setTimeout(() => resolve({ url: arg, data: {}}), 200)
});

const actions = createActions({
  fetchPage: {
    invocationType: 'async',
    type: 'PAGE',
    fn: async (url) => fetch(url),
  },
});

const created = actions(dispatch);

created.fetchPage('http://content.json');
```

Output:
```bash
{ type: 'PAGE_REQUESTED', payload: 'http://content.json' }
{ type: 'PAGE_SUCCEEDED',
  payload: { url: 'http://content.json', data: {} } }
{ type: 'PAGE_DONE' }
```

### Async Declarations - Overriding Errors
If there are cases in which we expect an error to occur and want to respond with a specific payload. We can add an `errorPayload` value during our declaration. The original error will be ignored and our error data will be dispatched.
```javascript

const actions = createActions({
  fetchPage: {
    invocationType: 'async',
    type: 'PAGE',
    errorTransform: (e) => ({
      error: e.message,
      override: true,
    }),
    fn: async () => {
      throw new Error('nope');
    },
  },
});

const created = actions(console.log);

created.fetchPage();
```

Output: 
```bash
// { type: 'PAGE_REQUESTED', payload: [] }
// { type: 'PAGE_FAILED',
//   payload: { error: 'nope', override: true } }
// { type: 'PAGE_DONE' }
```

### Synchronous Declarations - Simple
Automatically wraps your defined function response into a dispatch call.

```javascript
import { createActions } from '@sullivan/redux-config';

const dispatch = console.log;

const actions = createActions({
 autoDispatch: (id) => ({
    type: 'VIEW_ID',
    payload: {
      id, 
    }
  }),
});

const created = actions(dispatch);

created.autoDispatch('123');
```

Output:
```bash
{ type: 'VIEW_ID', payload: { id: '123' } }
  ```

### Synchronous Declarations - Static
Creates a static action that does not receive params. 

```javascript
import { createActions } from '@sullivan/redux-config';

const dispatch = console.log;

const actions = createActions({
  navigate: {
    type: 'NAVIGATE',
    payload: {
      content: 'some-static-payload',
    }
  },
});

const created = actions(dispatch);

created.navigate();
```

Output:
```bash
{ type: 'NAVIGATE',
  payload: { content: 'some-static-payload' } }
  ```
  
  ### Synchronous Declarations - Dynamic Payload
If payload is missing in a static declaration, params are automatically passed in as the payload. Single params will be injected as the payload, but n>1 will be converted into an array.

```javascript
import { createActions } from '@sullivan/redux-config';

const dispatch = console.log;

const actions = createActions({
  autoPayload: {
    type: 'ENABLE',
  },
});

const created = actions(dispatch);

created.autoPayload(1, 2, 3);
created.autoPayload({ foo: 'bar' });
```

Output:
```bash
{ type: 'ENABLE', payload: [ 1, 2, 3 ] }
{ type: 'ENABLE', payload: { foo: 'bar' } }
  ```


# actionCreator
Configure a single action for use - follow the same convention as `createActions`.

Options:

| key| value | required | default |
|--|--|--|--|
| type | string | yes | - |
| invocationType | string | - | 'sync' |
| fn | function | async only | - |
| errorTransform | AsyncFunction | - | - |
| payload | any | static only | - |
 

```javascript
import { actionCreator } from '@sullivan/redux-config';

const dispatch = console.log;

const fetch = (arg) => new Promise((resolve) => {
  setTimeout(() => resolve({ url: arg, data: {}}), 200)
});

const actions = actionCreator({
  invocationType: 'async',
  type: 'PAGE',
  fn: async (url) => fetch(url),
});

const fetchPage = actions(dispatch);

fetchPage('http://content.json');
```

Output:
```bash
{ type: 'PAGE_REQUESTED', payload: 'http://content.json' }
{ type: 'PAGE_SUCCEEDED',
  payload: { url: 'http://content.json', data: {} } }
{ type: 'PAGE_DONE' }
```
