# JavaScript Asynchronous Programming Quiz

### 1. Which of the following is a key difference between synchronous and asynchronous programming?

- A. Synchronous programming allows multiple tasks to run concurrently.

- B. Asynchronous programming executes tasks one after another, blocking the main thread.

- C. Asynchronous programming allows tasks to run without waiting for previous tasks to complete.

- D. Synchronous programming is always faster than asynchronous programming.

**Answer:** C

### 2. In the context of JavaScript, which of the following was the first approach to handling asynchronous operations?

- A. Promises

- B. Async/Await

- C. Callbacks

- D. Generators

**Answer:** C

### 3. What is "callback hell" in JavaScript?

- A. An error that occurs when a callback function fails

- B. A situation where callbacks are nested within other callbacks, leading to hard-to-read code

- C. A debugging technique for callbacks

- D. A security vulnerability related to callbacks

**Answer:** B

### 4. Which of the following methods in JavaScript returns a promise that resolves when all of the input promises have been fulfilled?

- A. Promise.race()

- B. Promise.any()

- C. Promise.all()

- D. Promise.resolve()

**Answer:** C

### 5. What are the possible states of a JavaScript Promise?

- A. Pending, Fulfilled, Rejected

- B. Start, In Progress, End

- C. Open, Closed, Error

- D. Initialized, Running, Completed

**Answer:** A

### 6. In JavaScript, which of the following keywords is used to pause the execution of an async function until a promise is settled?

- A. wait

- B. pause

- C. defer

- D. await

**Answer:** D

### 7. Which of the following is an advantage of using async/await over Promises with .then() and .catch()?

- A. Async/await allows writing asynchronous code in a synchronous style

- B. Async/await eliminates the need for error handling

- C. Async/await is faster than Promises

- D. Async/await does not rely on the event loop

**Answer:** A

### 8. When combining multiple Promises using Promise.all(), which of the following is true?

- A. The returned promise resolves as soon as any input promise resolves

- B. The returned promise rejects as soon as any input promise rejects

- C. The returned promise resolves when all input promises have resolved

- D. Both B and C

**Answer:** D

### 9. In the context of the eCommerce system example, which functions would benefit from being executed asynchronously?

- A. fetchItems

- B. createCsvFileForItems

- C. sendOrderStatus

- D. All of the above

**Answer:** D

### 10. What does the fetch() function in JavaScript return?

- A. A Response object

- B. A Promise

- C. JSON data

- D. All of the above

**Answer:** B

### 11. Which of the following methods is used to handle errors in a Promise chain?

- A. .then()

- B. .catch()

- C. .finally()

- D. throw()

**Answer:** B

### 12. Which statement is true about async functions in JavaScript?

- A. Async functions do not return anything

- B. Async functions always return a Promise

- C. Async functions block the main thread until completion

- D. All of the above

**Answer:** B

### 13. In promise terminology, when a promise is neither fulfilled nor rejected, it is said to be in which state?

- A. Settled

- B. Pending

- C. Resolved

- D. Inactive

**Answer:** B

### 14. Which of the following statements about Promise.any() is true?

- A. It resolves when any one of the input promises resolves

- B. It rejects only if all input promises reject

- C. It resolves with the value of the first fulfilled promise

- D. All of the above

**Answer:** D

### 15. Which of the following statements about async functions is true?

- A. They can use the await keyword to wait for promises to resolve

- B. They always return a Promise

- C. They allow asynchronous code to be written in a synchronous style

- D. All of the above

**Answer:** D
