# 👉 Check HTTP Errors
>Let’s say that you want to build a nice function fetch() that wraps the famous library axios, and - for the purpose of better error handling - you also want to throw some custom errors in case things go south:

```js
// Import AXIOS library:
// https://www.npmjs.com/package/axios
const axios = require("axios");

/**
 * Define some custom errors as explained in the earlier paragraph:
 */ 

class FetchError extends Error {
  constructor(originalError) {
    super(originalError.message);
    this.name = "FetchError";
    this.originalError = originalError;
  }
}

class RequestError extends FetchError {
  constructor(...params) {
    super(...params);
    this.name = "RequestError";
  }
}

class ResponseError extends FetchError {
  constructor(...params) {
    super(...params);
    this.name = "ResponseError";
  }
}

class NotFoundError extends ResponseError {
  constructor(...params) {
    super(...params);
    this.name = "NotFoundError";
  }
}
```

### The piece of code where you do the error handling is quite simple to identify:

```js
const fetch = async (url) => {
  try {
    const { data: { title } } = await axios.get(url);
    return `Todo: ${title}`;
  } catch (err) {
    // --->
    // HANDLE ERROR HERE!
    // <--- 
  }
};
```

### Without try...catch you may need to resolve to nested conditionals which is quite a poor way to handle this problem:

```js
/**
 * ❌ bad way, using nested conditionals:
 */

} catch (err) {
  if (err.response) {
    if (err.response.status === 404) {
      throw new NotFoundError(err);
    }
    throw new ResponseError(err);
  }
  if (err.request) {
    throw new RequestError(err);
  }
  throw new FetchError(err);
}
```

### But you can make a clever utilization of try...catch and SRP and build a function that identifies which error to throw in a more readable way:
```js
/**
 * ✅ good way, using try...catch and SRP:
 */

// Specialized function that identifies the error thrown by Axios:
const getFetchError = (err) => {
  // Test for a Request error:
  try {
    if (err.response.status === 404) {
      return new NotFoundError(err);
    }
    return new ResponseError(err);
  } catch (err) {}

  // Test for a Response error:
  try {
    if (err.request) {
      return new RequestError(err);
    }
  } catch (err) {}

  // Fallback on a generic error:
  return new FetchError(err);
};

// Here goes the entire `fetch()` implementation:
const fetch = async (url) => {
  try {
    const { data: { title } } = await axios.get(url);
    return `Todo: ${title}`;
  } catch (err) {
    throw getFetchError(err);
  }
};
```

### Even if for this simplistic example the “good way” means writing more code, we still achieve:

- better readability
- better unit testing
(as we can test the getFetchError() function in complete isolation.)
- fulfillment of SRP

#js #tip 