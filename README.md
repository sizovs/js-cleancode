### Follow "The Stepdown Rule"

Organize your functions in a file according to the stepdown rule. Higher level functions should be on top and lower levels below. It makes it natural to read the source code.

**Bad:**

```javascript
// "I need the full name for something..."
function getFullName (user) {
  return `${user.firstName} ${user.lastName}`
}

function renderEmailTemplate (user) {
  // "oh, here"
  const fullName = getFullName(user)
  return `Dear ${fullName}, ...`
}
```

**Good:**

```javascript
function renderEmailTemplate (user) {
  // "I need the full name of the user"
  const fullName = getFullName(user)
  return `Dear ${fullName}, ...`
}

// "I use this for the email template rendering"
function getFullName (user) {
  return `${user.firstName} ${user.lastName}`
}
```

### Use promises instead of callbacks
Callbacks aren't clean, and they cause excessive amounts of nesting. With ES2015/ES6, Promises are a built-in global type. Use them!

**Bad:**

```javascript
import { get } from "request";
import { writeFile } from "fs";

get(
  "https://en.wikipedia.org/wiki/Robert_Cecil_Martin",
  (requestErr, response) => {
    if (requestErr) {
      console.error(requestErr);
    } else {
      writeFile("article.html", response.body, writeErr => {
        if (writeErr) {
          console.error(writeErr);
        } else {
          console.log("File written");
        }
      });
    }
  }
);
```

**Good:**

```javascript
import { get } from "request";
import { writeFile } from "fs";

get("https://en.wikipedia.org/wiki/Robert_Cecil_Martin")
  .then(response => {
    return writeFile("article.html", response);
  })
  .then(() => {
    console.log("File written");
  })
  .catch(err => {
    console.error(err);
  });
```

### Async/Await are even cleaner than Promises

**Bad:**
```javascript
import { get } from "request-promise";
import { writeFile } from "fs-promise";

get("https://en.wikipedia.org/wiki/Robert_Cecil_Martin")
  .then(response => {
    return writeFile("article.html", response);
  })
  .then(() => {
    console.log("File written");
  })
  .catch(err => {
    console.error(err);
  });
```

**Good:**

```javascript
import { get } from "request-promise";
import { writeFile } from "fs-promise";

async function getCleanCodeArticle() {
  try {
    const response = await get(
      "https://en.wikipedia.org/wiki/Robert_Cecil_Martin"
    );
    await writeFile("article.html", response);
    console.log("File written");
  } catch (err) {
    console.error(err);
  }
}
```

### Don't ignore caught errors

Doing nothing with a caught error doesn't give you the ability to ever fix
or react to said error. Logging the error to the console (`console.log`)
isn't much better as often times it can get lost in a sea of things printed
to the console. If you wrap any bit of code in a `try/catch` it means you
think an error may occur there and therefore you should have a plan,
or create a code path, for when it occurs.

**Bad:**

```javascript
try {
  functionThatMightThrow();
} catch (error) {
  console.log(error);
}
```

**Good:**

```javascript
try {
  functionThatMightThrow();
} catch (error) {
  // One option (more noisy than console.log):
  console.error(error);
  // Another option:
  notifyUserOfError(error);
  // Another option:
  reportErrorToService(error);
  // OR do all three!
}
```

### Don't ignore rejected promises

For the same reason you shouldn't ignore caught errors
from `try/catch`.

**Bad:**

```javascript
getdata()
  .then(data => {
    functionThatMightThrow(data);
  })
  .catch(error => {
    console.log(error);
  });
```

**Good:**

```javascript
getdata()
  .then(data => {
    functionThatMightThrow(data);
  })
  .catch(error => {
    // One option (more noisy than console.log):
    console.error(error);
    // Another option:
    notifyUserOfError(error);
    // Another option:
    reportErrorToService(error);
    // OR do all three!
  });
```

### Reduce side-effects

Use pure functions without side effects, whenever you can. They are really easy to use and test.

**Bad:**

```javascript
function addItemToCart (cart, item, quantity = 1) {
  const alreadyInCart = cart.get(item.id) || 0
  cart.set(item.id, alreadyInCart + quantity)
  return cart
}
```

**Good:**
```javascript
function addItemToCart (cart, item, quantity = 1) {
  const cartCopy = new Map(cart)
  const alreadyInCart = cartCopy.get(item.id) || 0
  cartCopy.set(item.id, alreadyInCart + quantity)
  return cartCopy
}
```

### Only comment things that have business logic complexity.

In-line comments should be used sparingly, only where the code is not "self-documenting".

**Bad:**

```javascript
function hashIt(data) {
  // The hash
  let hash = 0;

  // Length of string
  const length = data.length;

  // Loop through every character in data
  for (let i = 0; i < length; i++) {
    // Get character code.
    const char = data.charCodeAt(i);
    // Make the hash
    hash = (hash << 5) - hash + char;
    // Convert to 32-bit integer
    hash &= hash;
  }
}
```

**Good:**

```javascript
function hashIt(data) {
  let hash = 0;
  const length = data.length;

  for (let i = 0; i < length; i++) {
    const char = data.charCodeAt(i);
    hash = (hash << 5) - hash + char;

    // Convert to 32-bit integer
    hash &= hash;
  }
}
```

### Single concept per test

Prefer many small and specific tests over one-size-fits-all monster test.

**Bad:**

```javascript
import assert from "assert";

describe("MakeMomentJSGreatAgain", () => {
  it("handles date boundaries", () => {
    let date;

    date = new MakeMomentJSGreatAgain("1/1/2015");
    date.addDays(30);
    assert.equal("1/31/2015", date);

    date = new MakeMomentJSGreatAgain("2/1/2016");
    date.addDays(28);
    assert.equal("02/29/2016", date);

    date = new MakeMomentJSGreatAgain("2/1/2015");
    date.addDays(28);
    assert.equal("03/01/2015", date);
  });
});
```

**Good:**

```javascript
import assert from "assert";

describe("MakeMomentJSGreatAgain", () => {
  it("handles 30-day months", () => {
    const date = new MakeMomentJSGreatAgain("1/1/2015");
    date.addDays(30);
    assert.equal("1/31/2015", date);
  });

  it("handles leap year", () => {
    const date = new MakeMomentJSGreatAgain("2/1/2016");
    date.addDays(28);
    assert.equal("02/29/2016", date);
  });

  it("handles non-leap year", () => {
    const date = new MakeMomentJSGreatAgain("2/1/2015");
    date.addDays(28);
    assert.equal("03/01/2015", date);
  });
});
```
