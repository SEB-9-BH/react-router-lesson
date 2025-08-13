# React Router Bitcoin PriceFinder — Vite Edition

---

We are often used to making websites with several "pages" which would be split across several html delivered statically or rendered by templates on a server. When making a React app, the application is a single page with one html file. We can have components conditionally render to make the illusion of pages but it doesn't quite feel as intuitive as using a tags to link to different html files.

What the React-Router library does is allow us to define components that render based on the url in the address bar. We link to them with Link components which feel similar to the a tags we are used to. It allows to create a single page application in a way that feels like a multi-page application.

## 1) Lesson Outcomes

* Understand `useEffect` deeply.
* Set up React Router in Vite.
* Build pages with URL params.
* Fetch live prices and render state.
* Follow our folder and import rules.

---

## 2) `useEffect`… line by line

**Rule.** Rendering stays pure. Effects handle side work.

```jsx
import { useEffect, useState } from "react";

export default function Demo({ url }) {
  const [data, setData] = useState(null);      // [1]
  const [loading, setLoading] = useState(false); // [2]
  const [error, setError] = useState("");        // [3]

  useEffect(() => {                               // [4]
    if (!url) return;                             // [5]
    const controller = new AbortController();     // [6]

    async function load() {                       // [7]
      try {
        setLoading(true);                         // [8]
        setError("");                             // [9]
        const res = await fetch(url, { signal: controller.signal }); // [10]
        if (!res.ok) throw new Error(`HTTP ${res.status}`);          // [11]
        const json = await res.json();            // [12]
        setData(json);                            // [13]
      } catch (e) {
        if (e.name !== "AbortError") setError(e.message); // [14]
      } finally {
        setLoading(false);                        // [15]
      }
    }

    load();                                       // [16]
    return () => controller.abort();              // [17]
  }, [url]);                                      // [18]

  return null;
}
```

**What each marker means.**
- [1] Data lives in state.
- [2] Loading flag controls UI.
- [3] Error string explains failures.
- [4] `useEffect` runs after paint.
- [5] Guard when input is empty.
- [6] Create an abort controller.
- [7] Define async worker inside effect.
- [8] Flip loading on.
- [9] Clear stale errors.
- [10] Fetch with abort signal.
- [11] Throw on non-2xx.
- [12] Parse JSON.
- [13] Commit data to state.
- [14] Ignore abort as an error.
- [15] Always end loading.
- [16] Start work.
- [17] Cancel if deps change or unmount.
- [18] Run on mount… and whenever `url` changes.

**Dev note.** StrictMode replays mount effects in dev. Production runs once.

---

## 3) Project Setup

```bash
npm create vite@latest CryptoPrices -- --template react
cd CryptoPrices
npm install
npm i react-router-dom
```

Create `.env` in the project root.

```env
VITE_COINAPI_KEY=YOUR_KEY_HERE
```

Vite exposes only keys that start with `VITE_`.

---

## 4) Folder Structure

```
src/
  main.jsx
  App.jsx
  index.css
  components/
    Nav/Nav.jsx
  pages/
    Main/Main.jsx
    Currencies/Currencies.jsx
    Price/Price.jsx
```

All names are capital.
Each component sits in its own folder.

---

## 5) Entry Point… line by line

The first component we'll explore is BrowserRouter which is underneath the hood a context provider allowing all the features of router to be available to its children. We want all of our application to have the router features so we'll wrap the App component in index.js and to make it more semantic we'll rename the component Router.

`src/main.jsx`

```jsx
import { StrictMode } from "react";                 // [1]
import { createRoot } from "react-dom/client";      // [2]
import { BrowserRouter as Router } from "react-router-dom"; // [3]
import App from "./App.jsx";                        // [4]
import "./index.css";                               // [5]

createRoot(document.getElementById("root")).render( // [6]
  <StrictMode>                                      // [7]
    <Router>                                        // [8]
      <App />                                       // [9]
    </Router>
  </StrictMode>
);
```

- [1] Use StrictMode in dev.
- [2] Use the modern root API.
- [3] Make routing available to children.
- [4] Import the app shell.
- [5] Global styles load once.
- [6] Bind React to the root div.
- [7] Strict checks catch issues.
- [8] Router watches the URL.
- [9] App renders inside Router.

---

## A common convention is to create two folders, components and pages. Any component that is used as a piece of UI goes in the components folder, any component meant to act as a "page" of the website goes in pages.

- create a components and pages folder
- create a Currencies, Main, Price folders and JSX in the pages folder
- create the component boilerplate in each component

Main

```js
import React from "react";

const Main = (props) => {
  return <h1>This is the Main Component</h1>;
};

export default Main;
```

Currencies

```js
import React from "react";

const Currencies = (props) => {
  return <h1>This is the Currencies Component</h1>;
};

export default Currencies;
```

Price

```js
import React from "react";

const Price = (props) => {
  return <h1>This is the Price Component</h1>;
};

export default Price;
```

## 6) App with Routes… line by line

`src/App.jsx`

```jsx
import { Routes, Route } from "react-router-dom";           // [1]
import Nav from "./components/Nav/Nav.jsx";                 // [2]
import Main from "./pages/Main/Main.jsx";                   // [3]
import Currencies from "./pages/Currencies/Currencies.jsx"; // [4]
import Price from "./pages/Price/Price.jsx";                // [5]
import "./App.css";                                         // [6]

export default function App() {
  return (
    <div className="App">                                   {/* [7] */}
      <Nav />                                               {/* [8] */}
      <Routes>                                              {/* [9] */}
        <Route path="/" element={<Main />} />               {/* [10] */}
        <Route path="/currencies" element={<Currencies />} />{/* [11] */}
        <Route path="/price/:symbol" element={<Price />} /> {/* [12] */}
      </Routes>
    </div>
  );
}
```

- [1] Bring in routing primitives.
- [2] Top navigation.
- [3–5] Page components.
- [6] Local styles if needed.
- [7] App wrapper.
- [8] Nav stays visible across pages.
- [9] Router outlet for page matching.
- [10] Home route.
- [11] Coin list route.
- [12] Dynamic route with a `:symbol` param.

---

## 7) Navigation… line by line

`src/components/Nav/Nav.jsx`

```jsx
import { Link } from "react-router-dom";  // [1]
import "./Nav.css";                       // [2] optional

export default function Nav() {
  return (
    <div className="nav">                 {/* [3] */}
      <Link to="/"><div>CRYPTO PRICES</div></Link>       {/* [4] */}
      <Link to="/currencies"><div>CURRENCIES</div></Link>{/* [5] */}
    </div>
  );
}
```

- [1] `Link` changes URL without reload.
- [2] Keep styles colocated if desired.
- [3] Simple bar layout.
- [4] Link to home.
- [5] Link to list.

Add to `src/index.css`:

```css
.nav { display: flex; justify-content: space-between; background: #000; color: #fff; padding: 15px; font-size: 2em; }
.nav a { color: #fff; text-decoration: none; }
```

---



## 8) Currencies Page… line by line

`src/pages/Currencies/Currencies.jsx`

```jsx
import { Link } from "react-router-dom";                        // [1]

export default function Currencies() {
  const currencies = [                                          // [2]
    { name: "Bitcoin", symbol: "BTC" },
    { name: "Litecoin", symbol: "LTC" },
    { name: "Ethereum", symbol: "ETH" },
    { name: "Ethereum Classic", symbol: "ETC" },
    { name: "Stellar Lumens", symbol: "XLM" },
    { name: "Dash", symbol: "DASH" },
    { name: "Ripple", symbol: "XRP" },
    { name: "Zcash", symbol: "ZEC" },
  ];

  return (
    <div className="currencies">
      {currencies.map(({ name, symbol }) => (                  // [3]
        <Link key={symbol} to={`/price/${symbol}`}>            {/* [4] */}
          <h2>{name}</h2>                                      {/* [5] */}
        </Link>
      ))}
    </div>
  );
}
```

- [1] We will link to each coin’s price page.
- [2] Whitelist of supported coins.
- [3] Map to elements.
- [4] Build a URL with the `symbol` param.
- [5] Render a readable name.

---

## 9) Price Page with `useEffect`… line by line

`src/pages/Price/Price.jsx`

```jsx
import { useParams } from "react-router-dom";   // [1]
import { useEffect, useState } from "react";    // [2]

export default function Price() {
  const { symbol } = useParams();               // [3]
  const apiKey = import.meta.env.VITE_COINAPI_KEY; // [4]

  const [coin, setCoin] = useState(null);       // [5]
  const [loading, setLoading] = useState(false);// [6]
  const [error, setError] = useState("");       // [7]

  const url = `https://rest.coinapi.io/v1/exchangerate/${symbol}/USD?apikey=${apiKey}`; // [8]

  useEffect(() => {                              // [9]
    if (!symbol) return;                         // [10]
    const controller = new AbortController();    // [11]

    async function fetchCoin() {                 // [12]
      try {
        setLoading(true);                        // [13]
        setError("");                            // [14]
        const res = await fetch(url, { signal: controller.signal }); // [15]
        if (!res.ok) throw new Error(`HTTP ${res.status}`);          // [16]
        const data = await res.json();           // [17]
        setCoin(data);                           // [18]
      } catch (err) {
        if (err.name !== "AbortError") setError(err.message); // [19]
      } finally {
        setLoading(false);                       // [20]
      }
    }

    fetchCoin();                                 // [21]
    return () => controller.abort();             // [22]
  }, [symbol, url]);                              // [23]

  if (loading) return <h1>Loading…</h1>;         // [24]
  if (error) return <h1>Error: {error}</h1>;     // [25]
  if (!coin) return <h1>No data</h1>;            // [26]

  return (
    <div>
      <h1>{coin.asset_id_base} / {coin.asset_id_quote}</h1>   {/* [27] */}
      <h2>{coin.rate}</h2>                                    {/* [28] */}
      <p>Symbol from URL: {symbol}</p>                        {/* [29] */}
    </div>
  );
}
```

- [1] Read params from the URL.
- [2] Use hooks via destructured imports.
- [3] Grab the `:symbol` segment.
- [4] Read the API key from Vite env.
- [5–7] Local state for UI.
- [8] Build the request URL.
- [9] Run an effect on mount and on param change.
- [10] Skip if param missing.
- [11] Prepare for cancellation.
- [12] Define async worker.
- [13] Start loading.
- [14] Clear old errors.
- [15] Fetch with signal.
- [16] Throw on bad status.
- [17] Parse the body.
- [18] Save data.
- [19] Only show real errors.
- [20] End loading.
- [21] Execute worker.
- [22] Cancel on change or unmount.
- [23] Depend on inputs that change.
- [24–26] Render by state branch.
- [27–29] Show result and echo the param.

---

## 10) Main Page

`src/pages/Main/Main.jsx`

```jsx
export default function Main() {
  return <h1>This is the Main Page Please Choose A Link To Go To Your Preferred Page</h1>;
}
```

Simple landing page for the home route.

---

## 11) How it feels like pages

* `Link` updates the URL… no reload.
* `Routes` selects which page to render.
* `useParams` reads the dynamic part of the URL.
* `useEffect` reacts to param changes and fetches.
* Setting state triggers a re-render… the UI updates itself.

---

## 12) Test Flow

1. Run `npm run dev`.
2. Click **Currencies**.
3. Click **Bitcoin**.
4. URL shows `/price/BTC`.
5. The price page fetches… then renders rate.

---

## 13) Quick Troubleshooting

* Blank page… confirm `Router` wraps `App`.
* Env missing… check `.env` name and restart dev server.
* CORS or HTTP errors… read the status in the error message.
* Double fetch in dev… StrictMode replay is expected.

---

## 14) Recap

* You learned the real lifecycle of `useEffect`.
* You wired routes with params in Vite.
* You fetched data safely with cleanup.
* You followed our folder rules and import pattern.


