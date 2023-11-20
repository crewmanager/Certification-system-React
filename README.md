# Certification-system-React

`main.mo`

```js

import HashMap "mo:base/HashMap";
import Principal "mo:base/Principal";
import List "mo:base/List";
import Iter "mo:base/Iter";
import Time "mo:base/Time";
import Debug "mo:base/Debug";

actor certification {

  private stable var registeredEntries : [(Principal, User)] = [];

  // private stable var savedList : [Principal] = [];

  type HashMap<K, V> = HashMap.HashMap<K, V>;

  stable var registedOnesList : List.List<Principal> = List.nil<Principal>();

  var registedUsersHashmap : HashMap<Principal, User> = HashMap.HashMap<Principal, User>(5, Principal.equal, Principal.hash);

  let certificateHolders : HashMap<Principal, Certification> = HashMap.HashMap<Principal, Certification>(5, Principal.equal, Principal.hash);

  // Registeration
  type User = {
    name : Text;
    isRegistered : Bool;
  };

  type Certification = {
    name : Text;
    issueTimestamp : Int;
    isRevoked : Bool;
  };

  public shared (msg) func registerUser(namee : Text) : async Text {

    let obj : User = { name = namee; isRegistered = true };

    let userId : Principal = msg.caller;

    var iter = Iter.fromList(registedOnesList);

    for (user in iter) {
      if (user == userId) {
        return "you are already registered buddy!";
      };
    };

    registedOnesList := Iter.toList(iter);

    registedUsersHashmap.put(userId, obj);
    registedOnesList := List.push(userId, registedOnesList);

    return "success registering!!";

  };

  public func getCertificate(address : Principal) : async Text {

    var iter = Iter.fromList(registedOnesList);
    var userFound : Bool = false;

    let certiName : Text = "Blockchain Panjab Certificate";

    for (user in iter) {
      if (user == address) {

        let obj : Certification = {
          name = certiName;
          issueTimestamp = Time.now();
          isRevoked = false;
        };

        certificateHolders.put(address, obj);

        Debug.print(certiName # "Certificate has been issued!");
        userFound := true;
        // return "you are already registered buddy!";
      } 
    };

    if(userFound == false){
      return "user has not registered himself";
    };

    registedOnesList := Iter.toList(iter);

    return "Certificate has been issued!";

  };

  public shared(msg) func getId(): async Principal{
    return msg.caller;
  };

  public query func getUsers() : async List.List<Principal> {
    return registedOnesList;
  };

  system func preupgrade() {
    registeredEntries := Iter.toArray(registedUsersHashmap.entries());
  };

  system func postupgrade() {

    registedUsersHashmap := HashMap.fromIter<Principal, User>(registeredEntries.vals(), 1, Principal.equal, Principal.hash);


      
    // if (balances.size() < 1) {
    //   balances.put(owner, totalSupply);
    // };
  };

};
```
`main.css`

```css
body {
    font-family: sans-serif;
    font-size: 1.5rem;
}

img {
    max-width: 50vw;
    max-height: 25vw;
    display: block;
    margin: auto;
}

form {
    display: flex;
    justify-content: center;
    gap: 0.5em;
    flex-flow: row wrap;
    max-width: 40vw;
    margin: auto;
    align-items: baseline;
}

button[type="submit"] {
    padding: 5px 20px;
    margin: 10px auto;
    float: right;
}

#greeting {
    margin: 10px auto;
    padding: 10px 60px;
    border: 1px solid #222;
}

#greeting:empty {
    display: none;
}
```
`index.jsx`

```js

import React from "react";
import ReactDOM from "react-dom";
import App from "./components/App";

ReactDOM.render(<App />, document.getElementById("root"));
```
`App.jsx`

```js

import React from 'react';
import CertificateApp from './CertificateApp';


function App() {
  return (
    <div className="App">
      <CertificateApp/>
    </div>
  );
}

export default App;
```
`CertificateApp.jsx`

```js

import React, { useState } from 'react';
import './CertificateApp.css'; // Import your CSS file
import TextDisplay from './TextDisplay'; // Make sure to provide the correct path
import { certiSys_backend } from '../../../declarations/certiSys_backend';

function CertificateApp() {
  const [name, setName] = useState('');
  const [inputData, setInputData] = useState('');
  const [registeredMsg, setRegisteredMsg] = useState('');
  const [id, setId] = useState('');

  const handleNameChange = async (e) => {
    setName(e.target.value);
  };

  const handleInputDataChange = (e) => {
    setInputData(e.target.value);
  };

  const handleRegister = async () => {
    const regMsg = await certiSys_backend.registerUser(name);
    setRegisteredMsg(regMsg);
  };

  const getIdd = async () => {
    const id = await certiSys_backend.getId();
    setId(id);
  }

  return (
    <div className="container">
      <h1>Register Yourself for a Certification</h1>
      <div>
        <label>
          Enter Your Name:
          <input type="text" value={name} onChange={handleNameChange} />
        </label>
        <button onClick={handleRegister}>Register</button>
      </div>
      {registeredMsg && <div className="result">{registeredMsg}</div>}
      <div className="id-section">
        <h1>Get Your ID</h1>
        {id ? "your id is "+ id : (
          <button className="id-button" onClick={getIdd}>Get ID</button>
        )}
      </div>
      <TextDisplay className="text-display" />
    </div>
  );
}

export default CertificateApp;
```
`TextDisplay.jsx`

```js

import React, { useState } from 'react';
import { certiSys_backend } from '../../../declarations/certiSys_backend';
import {Principal} from '@dfinity/principal'

function TextDisplay() {
  const [inputId, setInputId] = useState('');
  const [displayText, setDisplayText] = useState('');

  const handleIdChange = async (e) => {
    setInputId(e.target.value);
   
  };

  const handleDisplayText = async () => {
    console.log(inputId)

    const message = await certiSys_backend.getCertificate(Principal.fromText(inputId));
    console.log(message);
    setDisplayText(message);
    console.log(displayText);
   
  };

  return (
    <div>
      <h1>Get your certificate !! </h1>
      <div>
        <label>
          Enter your ID :
          <input type="text" value={inputId} onChange={handleIdChange} />
        </label>
        <button onClick={handleDisplayText}>Get Certificate!</button>
      </div>
      {displayText && (
        <div>
          <p>Text Displayed: {displayText} </p>
        </div>
      )}
    </div>
  );
}

export default TextDisplay;

```
`package.json`

```js
// inside package.json update depedencies

"dependencies": {
    "@dfinity/agent": "^0.19.3",
    "@dfinity/candid": "^0.19.3",
    "@dfinity/principal": "^0.19.3",
    "@emotion/react": "^11.11.1",
    "@emotion/styled": "^11.11.0",
    "@material-ui/core": "^4.12.4",
    "@material-ui/icons": "^4.11.3",
    "@mui/icons-material": "^5.14.14",
    "@mui/material": "^5.14.14",
    "autoprefixer": "^10.4.16",
    "postcss": "^8.4.31",
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    "ts-loader": "^9.5.0"
  }

```

`tsconfig.json`

```js
//add this where webpack.config.js is present 

{
    "compilerOptions": {
      "target": "es2018",        /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019' or 'ESNEXT'. */
      "lib": ["ES2018", "DOM"],  /* Specify library files to be included in the compilation. */
      "allowJs": true,           /* Allow javascript files to be compiled. */
      "jsx": "react",            /* Specify JSX code generation: 'preserve', 'react-native', or 'react'. */
    },
    "include": ["src/**/*"]
  }
  
```

`index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Document</title>
    <base href="/" />
    <link rel="icon" href="favicon.ico" />
    <link type="text/css" rel="stylesheet" href="main.css" />
  	<link id="external-css" rel="stylesheet" type="text/css" href="https://fonts.googleapis.com/css?family=Montserrat&amp;display=swap" media="all">
</head>
  <body>
    <div id="root"></div>
  </body>
</html>

```
