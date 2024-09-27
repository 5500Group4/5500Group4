# Analyzing Client Server Software in NodeJS and React TypeScript

### Due date: Sep 27 by 5:59 PM

This document includes the analyses of the [Calc Sheet App example](https://classroom.github.com/a/ztxdcNowLinks).



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