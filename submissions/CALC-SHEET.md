# Analyzing Client Server Software in NodeJS and React TypeScript

### Due date: Sep 27 by 5:59 PM

This document includes the analyses of the [Calc Sheet App example](https://classroom.github.com/a/ztxdcNowLinks).

## Part 2: Analyzing the Backend (NodeJS)

### 1. Examine the RESTful API:

Explore how the backend is structured to serve API requests. Identify key API endpoints that the client interacts with and document:

- #### a. How data is created, read, updated, and deleted (CRUD operations).

**Answer:**

The program uses Express JS to build up the backend framework, which completes the routing based on HTTP method and URL pattern. This backend API is designed to handle operations on documents, with a focus on CRUD functionality (Create, Read, Update, Delete) for both the documents and document cells. Here’s how the API is structured to serve client requests by different API endpoints:

- **Create:**
  - Create a new document containing a new spreadsheet (the file has two endpoints for creation).


    ```typescript
    app.put('/documents/:name', (req: express.Request, res: express.Response) => {
    .....
        const documentNames = documentHolder.getDocumentNames();
        if (documentNames.indexOf(name) === -1) {
            console.log(`Document ${name} not found, creating it`);
            documentHolder.createDocument(name, 5, 8, userName);
        }
        const document = documentHolder.getDocumentJSON(name, userName);
        res.status(200).send(document);
    });
    ```

    ```typescript
    app.post('/documents/create/:name', (req: express.Request, res: express.Response) => {
    .....
	    const documentNames = documentHolder.getDocumentNames();
	    if (documentNames.indexOf(name) === -1) {
	        const documentOK = documentHolder.createDocument(name, 5, 8, userName);
	    }
	    documentHolder.requestViewAccess(name, 'A1', userName);
	    const documentJSON = documentHolder.getDocumentJSON(name, userName);
	
	    res.status(200).send(documentJSON);
    });
    ```

- **Read:**
  - Retrieve all document names.  
    
    ```typescript
    app.get('/documents', (req: express.Request, res: express.Response) => {
        const documentNames = documentHolder.getDocumentNames();
        res.send(documentNames);
    });
    ```

- **Update:**
  - Update Document Cell by adding a token or a cell

    ```typescript
    app.put('/document/addtoken/:name', (req: express.Request, res: express.Response) => {
        // ......
        // add the token
        const resultJSON = documentHolder.addToken(name, token, userName);
    });
    ```

    ```typescript
    app.put('/document/addcell/:name', (req: express.Request, res: express.Response) => {
        // ......
        const resultJSON = documentHolder.addCell(name, cell, userName);
    });
    ```
  - Update the editing status
    ```typescript
    app.put('/document/cell/edit/:name', (req: express.Request, res: express.Response) => {
	    const name = req.params.name;
	
	    // is this name valid?
	    const documentNames = documentHolder.getDocumentNames();
	    if (documentNames.indexOf(name) === -1) {
	        res.status(404).send(`Document ${name} not found`);
	        return;
	    }
	    // get the user name from the body
	    // get the cell from the body
	    const userName = req.body.userName;
	    const cell = req.body.cell;
	    if (!userName) {
	        res.status(400).send('userName is required');
	        return;
	    }
	    // request access to the cell
	    const result = documentHolder.requestEditAccess(name, cell, userName);
	    const documentJSON = documentHolder.getDocumentJSON(name, userName);
	
	    res.status(200).send(documentJSON);
	    });
    ```

- **Delete**
  - Delete the token in a document cell.

    ```typescript
    app.put('/document/removetoken/:name', (req: express.Request, res: express.Response) => {
        // ......
        const resultJSON = documentHolder.removeToken(name, userName);
    });
    ```

- #### b. How the server validates and processes client requests before responding.

**Answer:**

- **Validation:**
The API ensures required fields like userName, cell, and token are provided in the request body where applicable. It also validates document existence in most endpoints by checking the document names against the list held by DocumentHolder. For example, we note the the if-statements for the preliminary validation before processing client requests by the different API endpoints:

  - Validate the existence of userName in the `PUT /documents/:name`

    ```typescript
    if (!userName) {
	res.status(400).send('userName is required');
	return;
    }
    ```

  - Validate if the document name is within the list held by DocumentHolder in `PUT /document/addtoken/:name`:

    ```typescript
    if (documentNames.indexOf(name) === -1) {
	res.status(404).send(`Document ${name} not found`);
	return;
    }
    ```

- **Processing:**
Before responding to the client, the server interacts with the DocumentHolder class to create, update, or retrieve document data.
The server first create a new DocumentHolder class:
	```typescript
	const documentHolder = new DocumentHolder();
	```
	The endpoint `/documents` is used to retrieve the current documents in database:
 
	```typescript
	app.get('/documents', (req: express.Request, res: express.Response) => {
	    const documentNames = documentHolder.getDocumentNames();
	    res.send(documentNames);
	});
	```
  Additionally, the debug mode toggles logging of incoming requests to track client interactions for troubleshooting or development. The further operations such as adding token to the formula will be processed by the respective method of DocumentHolder object. For example, adding token is by ```documentHolder.addToken(name, token, userName)```. Adding a cell is by ```documentHolder.addCell(name, cell, userName);```.

### 2. Real-Time Communication (if applicable):

If the project includes real-time communication (e.g., using WebSockets), investigate:

#### a. How the client and server establish real-time connections.
#### b. How messages are exchanged between the client and server, and what kind of data is sent.

**Answer:**

The project currently relies on the HTTP request-response structure to build up the real-time communications between the client and the server.  

On the client side, the `SpreadSheetClient.ts` file manages the client’s connection and interaction with the server. The `setServerSelector` method is designed to determine the server address of the current operation. 

```typescript
setServerSelector(server: string): void {
  if (server === this._server) {
	return;
   }
  if (server === 'localhost') {
	this._baseURL = `${LOCAL_SERVER_URL}:${this._serverPort}`;
   } else {
	this._baseURL = RENDER_SERVER_URL;
   }
   this.getDocument(this._documentName, this._userName);
   this._server = server;
}
```

The _timedFetched() method is designed to fetch the document information at every 0.1 second and the document list at every 2 seconds from the server, so as to stimulate real time communication. 

```typescript
private async _timedFetch(): Promise<Response> {

// only get the document list every 2 seconds
let documentListInterval = 20;
let documentFetchCount = 0;
return new Promise((resolve, reject) => {
    setTimeout(() => {
	this.getDocument(this._documentName, this._userName);
	documentFetchCount++;
	if (documentFetchCount > documentListInterval) {
	    documentFetchCount = 0;
	    this.getDocuments(this._userName);
	}
	this._timedFetch();
    }, 100);
});
}
```

Furthermore, the `SpreadSheetClient` contains a variety of methods to exchange the messages to the server. It pulls the information by `getDocument()` and `getDocuments()`. It sends the information of calculation by `addToken()`, `removeToken()`, `addCell()` and `clearFormula()`, etc. It updates the user editing status by `setEditStatus()`. 

On the server side, the server listens for requests from the client on various endpoints in the DocumentServer.ts file. The data might be `userName`, `cell`, or `token`, based on the nature of the operation. It receives the data in the request URL path( as request parameters) and the body, performs actions accordingly and returns the data in JSON format. 

If the project needs to incorporate real-time communication using WebSockets, here’s how we can structure and implement the client-server interaction, focusing on how the connections are established, how messages are exchanged, and what kind of data is transmitted:

- **Establishing Real-Time Connections**
    To enable WebSocket communication between the client and server, we can:
	- On the Server: Set up a WebSocket server to handle incoming WebSocket connections.
	- On the Client: Establish a WebSocket connection to the server, usually after the page loads or when a specific event occurs.

- **Message Exchange**
	- Initiating Communication: Once the connection is established, both the client and server can send and receive messages through WebSocket events (message, open, close).
	- Message Format: Typically, JSON objects are used to send structured data. Messages might contain information such as document changes, user actions, or updates to collaborative data.

### 3. User Management:
#### a. How is user authentication handled (e.g., via JWT, OAuth)?

**Answer:**

The project requires a user to enter a username at the login page, and the username will be further set in the SpreadSheetClient class and transmitted to the backend. However, the user authentication functionality is not established in the project. 

The project might use OAuth-based authentication (e.g., Google, GitHub login). OAuth handles third-party authorization, allowing users to authenticate with external services. In OAuth, a user would authenticate via a third-party provider, and the server would receive an access token from the provider to verify the user’s identity.

The user will first be redirected to the OAuth provider to authenticate the identity and finish the authorization, which will generate the access token. The backend server needs to complete the configuration according to the instructions by different providers. The access token is required for all the route handlers in the backend. In such a way, an authorization layer should be built up between the client and the server. 

#### b. How does the backend differentiate between user types (e.g., admin, regular user) and provide different levels of access?

**Answer:**

Once users are authenticated, their roles determine what they can access or modify in the system. Common roles include:

- Admin: Full access to all features (create, read, update, delete).
- Regular User: Limited access, perhaps restricted to their own documents or specific actions.

We can manage the different levels of access by modifying the routes. One option is to add an authentication middleware in the route, for example, `app.put(‘/document/addcell/:name', isAdmin, (res, req) => )`. Some of the routes are limited to the users of certain roles. Another option is checking the user role. Take the add token route as the example: 

```typescript
if (req.body.userRole == ‘certain-role’) {
	const resultJSON = documentHolder.addToken(name, token, userName);
	res.status(200).send(resultJSON);
}
else {
	res.status(403).send(‘No Permission');
}
```

#### c. What mechanisms are used to secure sensitive user data?

**Answer:**

Several mechanisms ensure that sensitive data, such as user credentials and personal information, is securely handled:

- **Password Hashing**:  
  Passwords should never be stored in plaintext. Instead, a strong hashing algorithm like **bcrypt** is used to hash passwords before saving them in the database.

- **Salting**:  
  Random data (a "salt") is added to the password before hashing. This process makes the password more resistant to **rainbow table attacks**.

- **Hashing**:  
  **bcrypt** generates a hashed password that cannot be reversed, ensuring the stored password is secure.


### 4. Middleware and Error Handling:

Investigate how the backend uses middleware to handle tasks like:

- Authentication and session management.
- Data validation and error handling.
Document examples from the code that show how middleware simplifies request handling.

**Answer:**

Middleware plays a crucial role in handling tasks like authentication, session management, data validation, and error handling. Middleware allows these tasks to be modularized, reducing redundancy and improving maintainability.

- **Authentication and Session Management**
    Middleware is often used to manage authentication in Express applications by verifying tokens or sessions for each incoming request.

- **Data Validation and Error Handling**
    Middleware can be used to validate incoming data and handle errors globally, ensuring that each request passes through a standard validation process and that errors are handled consistently across the application.

In the project, we noted the following middlewares:

```typescript
app.use(cors());
```

This is the Cross-origin Resource Sharing (CORS) middleware that allows accessing resources from domains of different origins. The application is allowed to respond to the external third party APIs.It also automatically handles the preflight requests by responding with the appropriate headers. 

```typescript
app.use(bodyParser.json())
```

This is a middleware function to parse the content in the request body. It handles the JSON payloads for the application to access information. 

```typescript
app.use((req, res, next) => {
    if (debug) {
        console.log(`${req.method} ${req.url}`);
    }
    next();
});
```

This is a custom middleware to display the request method and URL when the debug flag is turned on. It can be deemed as logging such information related with request. 



## Part 3: Analyzing the Frontend (React TypeScript)

### 1. Multi-Screen Navigation:

Examine how the React app handles navigation between different screens and UI components:

- a. How is routing set up (e.g., using React Router)?
- b. How does the app handle protected routes (i.e., only allowing certain users to access specific pages)?

**Answer:**

- a. No, the site is not using React Router. The navigation between pages is done by functions under `App.tsx`, in which the functions retrieve the spreadsheet’s name and navigate to a specific page for a certain document name.
- b. The function `App()` is responsible for getting a specific `DocumentName`. For example, if the end point of the page address is null, it will then navigate to the login page. Otherwise, it will navigate to the specific spreadsheet through recognizing the end point name.

```typescript
// If there is no document name point this thing at /document
if (documentName === '') {
    setDocumentName('documents');
    resetURL('documents');
}

if (documentName === 'documents') {
    return (
        <div className="LoginPage">
            <header className="Login-header">
                <LoginPageComponent spreadSheetClient={spreadSheetClient} />
            </header>
        </div>
    );
}
```

### 2. State Management:

Investigate how the app manages state across different screens and components:

- a. How is user state (e.g., authentication status, role) maintained and shared between components?
- b. Are tools like Redux or Context API used to manage global state?

**Answer:**

- a. Under the `LoginPageComponent.tsx` file, it uses `LoginPageComponent()` to get the username and store it in `sessionStorage` to maintain the user state, and use functions like `loadDocument` to share between components.
- b. No, tools like Redux or Context API are not used to manage global state. Instead, it uses functions in `spreadSheetClient.tsx` to manage state. For example, the functions `getEditStatus()` and `getWorkingCellLabel()` to fetch data and share with the backend.

```typescript
function getUserLogin() {
    return <div>
        <input
            type="text"
            placeholder="User name"
            defaultValue={userName}
            onKeyDown={(event) => {
                if (event.key === 'Enter') {
                    // get the text from the input
                    let userName = (event.target as HTMLInputElement).value;
                    window.sessionStorage.setItem('userName', userName);
                    // set the user name
                    setUserName(userName);
                    spreadSheetClient.userName = userName;
                }
            }} 
        />
    </div>;
}
```

### 3. API Interaction:

Analyze how the frontend communicates with the backend:

- a. How are API calls made (e.g., using fetch, axios) and how is the data from the backend processed and displayed in the UI?
- b. How is the client updated so that they can see other users updating the cells?

**Answer:**

- a. In `spreadSheetClient.tsx`, it uses `fetch` to get responses from the frontend and pass them to the backend. For example, in function `removeToken()`, it uses `fetch` to get the username and pass it to the `_updateDocument()` function to keep the client internal document state up to date.
- b. When displaying data, the `spreadSheet` component uses `useEffect` to update the component’s state based on the latest data from `spreadSheetClient.tsx`. The `spreadSheetClient.tsx` keeps fetching the latest document data from the backend using the `_timedFetch()` function. Then the `useEffect` function in `SpreadSheet` will update the information from the backend.

```typescript
useEffect(() => {
    const interval = setInterval(() => {
        updateDisplayValues();
    }, 50);
    return () => clearInterval(interval);
});
```


### 4. User Interface:

Investigate how the app displays different UI components based on user roles:

- a. How does the frontend handle the display of real-time data if applicable (e.g., chat messages, notifications)?
- b. How is the cell ownership displayed to the users?

**Answer:**

- a. The `useEffect()` hook in the `SpreadSheetClient.tsx` is responsible for updating the display value every 1/20 second for all the components on screen.
- b. The interface `StatusProps` in `Status.tsx` is responsible for updating the `userName` for the `statusString` for cells. The `getSheetDisplayStringsForGUI()` function in `SpreadsheetClients.ts` compiles cell values and is formatted in the cell. Then the `SheetHolder()` in `SheetHolder.tsx` is responsible for displaying the cell value for users.

```typescript
public getSheetDisplayStringsForGUI(): string[][] {
    if (!this._document) {
        return [];
    }

    const columns = this._document.columns;
    const rows = this._document.rows;
    const cells: Map<string, CellTransport> = this._document.cells as Map<string, CellTransport>;
    const sheetDisplayStrings: string[][] = [];

    // create a 2d array of strings that is [row][column]
}

```


## Part 4: Frontend and Backend Interaction

### 1. API Request-Response Flow:

Trace the flow of data from the moment the frontend sends a request to the server to when it receives a response:

- a. Identify key points in the code where data flows from the server to the client and vice versa.
- b. How does the frontend handle errors returned by the server?

**Answer:**


```typescript
function onButtonClick(event: React.MouseEvent<HTMLButtonElement>): void {
    if (!checkUserName()) {
        return;
    }
    const text = event.currentTarget.textContent;
    let trueText = text ? text : "";
    spreadSheetClient.setEditStatus(true);
    spreadSheetClient.addToken(trueText);

    updateDisplayValues();
}

```

- a. First, in `SpreadSheet.tsx`, the function `onButtonClick()` and `onCellClick()` handles requests from clients. Then, the front-end sends the requests to the server as the event handlers in `SpreadSheetClient.ts` get the requests and use `fetch` to send the requests to the servers via methods such as `addToken`, `addCell`, `removeToken`, and `clearFormula`.


```typescript
public setEditStatus(isEditing: boolean): void {
    // request edit status of the current cell
    const body = {
        "userName": this._userName,
        "cell": this._document.currentCell
    };

    let requestEditViewURL = `${this._baseURL}/document/cell/view/${this._documentName}`;
    if (isEditing) {
        requestEditViewURL = `${this._baseURL}/document/cell/edit/${this._documentName}`;
    }

    fetch(requestEditViewURL, {
        method: 'PUT',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(body)
    });
}
```


```typescript
public addToken(token: string): void {
    const body = {
        "userName": this._userName,
        "token": token
    };

    const requestAddTokenURL = `${this._baseURL}/document/addtoken/${this._documentName}`;
    fetch(requestAddTokenURL, {
        method: 'PUT',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(body)
    })
    .then(response => {
        return response.json() as Promise<DocumentTransport>;
    });
}

```

The `then(response)` in `spreadsheet` will update the document in `_updateDocument()` which updates the client’s document.


```typescript
private _updateDocument(document: DocumentTransport): void {
    const formula = document.formula;
    const result = document.result;
    const currentCell = document.currentCell;
    const columns = document.columns;
    const rows = document.rows;
    const isEditing = document.isEditing;
    const contributingUsers = document.contributingUsers;
    const errorOccurred = document.errorOccurred;

    // create the document
    this._document = {
        formula: formula,
        result: result,
        currentCell: currentCell,
        columns: columns,
        rows: rows,
        isEditing: isEditing,
        contributingUsers: contributingUsers,
        errorOccurred: errorOccurred
    };
}
```

The `useEffect` in `SpreadSheet.tsx` consistently updates the states and data from `SpreadSheetClient.tsx`.


```typescript
// useEffect to refetch the data every 1/20 of a second
useEffect(() => {
    const interval = setInterval(() => {
        updateDisplayValues();
    }, 50);
    return () => clearInterval(interval);
});
```

And function `updateDisplayValues()` in `SpreadSheet.tsx` also updates component status from `SpreadSheetClient.tsx`.


```typescript
function updateDisplayValues(): void {
    spreadSheetClient.userName = userName;
    spreadSheetClient.documentName = documentName;

    setFormulaString(spreadSheetClient.getFormulaString());
    setResultString(spreadSheetClient.getResultString());
    setStatusString(spreadSheetClient.getEditStatusString());
    setCells(spreadSheetClient.getSheetDisplayStringsForGUI());
    setCurrentCell(spreadSheetClient.getWorkingCellLabel());
    setCurrentlyEditing(spreadSheetClient.getEditStatus());
}

```

- b. As for error handling, there is no proper catch and handle process in this project, but only an error display function.

For example, an `_errorCallback` function is called in `_updateDocuments()` in `SpreadSheetClient.tsx` and will call `displayErrorMessage()` in `App.tsx`.


```typescript
class SpreadSheetClient {
    private _updateDocument(document: DocumentTransport): void {
        const cell: CellTransport = {
            value: cellTransport.value,
            error: cellTransport.error,
            editing: this._getEditorString(contributingUsers, cellName)
        };
        this._document!.cells.set(cellName, cell);

        if (errorOccurred !== '') {
            this._errorCallback(errorOccurred);
        }
    }
}
```


```typescript
// callback for the error message
function displayErrorMessage(message: string) {
    alert(message);
}

function App() {
    const [documentName, setDocumentName] = useState(getDocumentNameFromWindow());
    const spreadSheetClient = new SpreadSheetClient('documents', '', displayErrorMessage);

    useEffect(() => {
        if (window.location.href) {
            setDocumentName(getDocumentNameFromWindow());
        }
    }, [getDocumentNameFromWindow]);
}
```

### 2. Real-Time Interaction (if applicable):

If the project includes real-time communication, investigate how the client and server communicate in real-time:

#### How are updates pushed from the server to the client, and how does the UI handle them?


**Answer:**

As discussed above, the project utilizes the HTTP request and response model to communicate between the client side and the server.  The functions to push the updates are placed within the `SpreadSheetClient.ts` file. The `_timedFetched()` method is designed to fetch the document information at every 0.1 second and the document list at every 2 seconds from the server. 

The updates to calculation content are pushed to the backend by different API endpoints. In the server, A `DocumentHolder` object will be created to take in the parameters such as token, cell and process the calculation. As is shown in the following `DocumentHolder`. `addToken()`, the result will be returned in a JSON format. 

```sh
     public addToken(docName: string, token: string, user: string,): any {
        let document = this._documents.get(docName);

        document!.addToken(token, user);
        this._saveDocument(docName);
        // get the json string for the controler
        const documentJSON = this.getDocumentJSON(docName, user);
        return documentJSON;

    }
```

The following code snippet is an example of pushing updates back to the client. The result is included on the response body and further sent by `res.status().send()`. 

```sh
    app.put('/document/addtoken/:name', (req: express.Request, res: express.Response) => {
        // ......
        const resultJSON = documentHolder.addToken(name, token, userName);
        res.status(200).send(resultJSON);
    }
```

In the `SpreadSheetClient.ts`, the UI displays results by the following method. A 2D array of string is generated to display the values in the cells, formatted for use in GUI.

```sh

   public getSheetDisplayStringsForGUI(): string[][] {
        if (!this._document) {
            return [];
        }
        const columns = this._document.columns;
        const rows = this._document.rows;
        const cells: Map<string, CellTransport> = this._document.cells as Map<string, CellTransport>;
        const sheetDisplayStrings: string[][] = [];
        // create a 2d array of strings that is [row][column]

        for (let row = 0; row < rows; row++) {
            sheetDisplayStrings[row] = [];
            for (let column = 0; column < columns; column++) {
                const cellName = Cell.columnRowToCell(column, row)!;
                const cell = cells.get(cellName) as CellTransport;
                if (cell) {
                    // add the cell value and the editing status
                    sheetDisplayStrings[row][column] = this._getCellValue(cell) + "|" + cell.editing;
                } else {
                    throw new Error(`cell ${cellName} not found`);
                }
            }
        }
        return sheetDisplayStrings;
    }
```
