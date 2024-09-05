## Step 1: Create a React Application
Run the following command to create a React application and save it in the folder `timegraph-app`:
```
npx create-react-app timegraph-app
cd timegraph-app
```
## Step 2: Install Necessary Packages
```
npm install @polkadot/extension-dapp @analog-labs/timegraph-js dotenv
```
## Step 3: Create a .env File
Replace `your-session-key-here` with your session key (this is a one-time setup):
```
REACT_APP_SESSION_KEY=your-session-key-here
```
## Step 4: Configure the React Application
Edit the `src/App.js` file with the content below
```javascript
import React, { useState, useRef } from "react";
import { TimegraphClient } from "@analog-labs/timegraph-js";
import { web3Enable } from "@polkadot/extension-dapp";

const sessionKey = process.env.REACT_APP_SESSION_KEY;
const timegraphGraphqlUrl = "https://timegraph.testnet.analog.one/graphql";

async function watchSDKTesting(name, hashId) {
  await web3Enable("abcd");

  const client = new TimegraphClient({
    url: timegraphGraphqlUrl,
    sessionKey: sessionKey, 
  });

  // Add fund
  const fund = await client.tokenomics.sponsorView({
    viewName: name,
    amount: "500000000"
  });

  console.log(fund);

  // Add alias
  let aliasResponse = await client.alias.add({
    name: name,
    hashId: hashId,
    identifier: name,
  });

  console.log(aliasResponse);

  // Get data
  const data = await client.view.data({
    _name: name, 
    hashId: hashId, 
    fields: ["_index"],
    limit: 10,
  });

  return { data, aliasResponse };
}

function App() {
  const [data, setData] = useState(null);
  const [aliasResponse, setAliasResponse] = useState(null); 
  const [fields, setFields] = useState([{ name: "", hashId: "" }]);
  const aliasResponseRef = useRef(null);

  const handleChange = (index, event) => {
    const values = [...fields];
    values[index][event.target.name] = event.target.value;
    setFields(values);
  };

  const handleAdd = () => {
    setFields([...fields, { name: "", hashId: "" }]);
  };

  const handleRemove = (index) => {
    const values = [...fields];
    values.splice(index, 1);
    setFields(values);
  };

  const handleSubmit = async (e) => {
    e.preventDefault(); 
    const promises = fields.map(({ name, hashId }) => watchSDKTesting(name, hashId));
    const results = await Promise.all(promises);
    setData(results.map(({ data }) => data));
    setAliasResponse(results.map(({ aliasResponse }) => aliasResponse));
    aliasResponseRef.current.scrollIntoView({ behavior: "smooth" });
  };

  return (
    <div>
      <h1>Timegraph Data</h1>
      
      <form onSubmit={handleSubmit}>
        {fields.map((field, index) => (
          <div key={index}>
            <label>
              Name:
              <input
                type="text"
                name="name"
                value={field.name}
                onChange={(e) => handleChange(index, e)}
                required
              />
            </label>
            <label>
              Hash ID:
              <input
                type="text"
                name="hashId"
                value={field.hashId}
                onChange={(e) => handleChange(index, e)}
                required
              />
            </label>
            <button type="button" onClick={() => handleRemove(index)}>Remove</button>
          </div>
        ))}
        <button type="button" onClick={handleAdd}>Add</button>
        <button type="submit">Submit</button>
      </form>

      {aliasResponse && (
        <div ref={aliasResponseRef}>
          <h2>Alias Response</h2>
          {aliasResponse.map((response, index) => (
            <pre key={index}>
              {JSON.stringify(
                {
                  status: response.status,
                  name: response.view.name,
                  description: response.view.description,
                  sql: response.view.sql,
                },
                null,
                2
              )}
            </pre>
          ))}
        </div>
      )}

      {data ? (
        <div>
          <h2>Timegraph Data</h2>
          {data.map((dataObj, index) => (
            <pre key={index}>{JSON.stringify(dataObj, null, 2)}</pre>
          ))}
        </div>
      ) : (
        <p>Loading data...</p>
      )}
    </div>
  );
}

export default App;
```
## Step 5: Run the Application
```
npm start
```
## Step 6: Access the Application to Run the Code
Typically, the app will be available at `http://localhost:3000`. 
You just need to open a browser and enter the address `http://localhost:3000` to run the query for you.

Now i need input name and hashId then submit. Done
