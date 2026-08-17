# React Redux Toolkit (RTK) & RTK Query: Zero to Hero
## Part 2: Redux Toolkit (RTK) Basics

---

# 📚 PART 2: REDUX TOOLKIT (RTK) BASICS

---

## 📚 MODULE 4: RTK Kya Hai, configureStore — Setup, Middleware, DevTools

---

### 🧠 4.1 Concept Explanation — RTK Ki Introduction

Redux Toolkit ek **official, opinionated toolset** hai Redux ke liye. "Opinionated" matlab: RTK ne decide kar liya hai ki Redux kaise use karna chahiye — best practices already built-in hain.

**RTK install karna:**
```bash
npm install @reduxjs/toolkit react-redux
```

**Do packages:**
1. `@reduxjs/toolkit` — RTK ki core library
2. `react-redux` — React components ko Redux store se connect karta hai

---

### 🌍 4.2 Real-World Analogy — Ready-Made Kitchen vs Raw Ingredients

**Plain Redux** = Raw ingredients khareedo (flour, eggs, sugar...) aur khud sab banaao.

**Redux Toolkit** = Ready-made kit milo jisme:
- Ingredients pehle se measured hain ✅
- Recipe already included hai ✅
- Common mistakes se protection hai ✅
- Best tools already included hain ✅

Sirf cooking karo, ingredient gathering nahi!

---

### 💻 4.3 configureStore — Deep Dive

`configureStore` Redux Toolkit ka store creation function hai.

```javascript
// store.js — Production-ready store setup
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './features/counter/counterSlice';
import cartReducer from './features/cart/cartSlice';
import userReducer from './features/user/userSlice';

const store = configureStore({
  // ---- reducer ----
  // Yahan sab feature reducers aate hain
  // Key naam = state mein is slice ka path
  reducer: {
    counter: counterReducer,  // state.counter
    cart: cartReducer,         // state.cart
    user: userReducer,         // state.user
  },

  // ---- middleware ----
  // By default: redux-thunk add hota hai
  // Custom middleware add karna:
  middleware: (getDefaultMiddleware) => 
    getDefaultMiddleware().concat(myCustomMiddleware),
  // getDefaultMiddleware() = [thunk, immutabilityMiddleware, serializabilityMiddleware]

  // ---- devTools ----
  // Development mein automatically enabled (process.env.NODE_ENV check karta hai)
  // Production mein automatically disabled
  devTools: process.env.NODE_ENV !== 'production',

  // ---- preloadedState ----
  // Initial state inject karna (e.g., from localStorage or server-side)
  preloadedState: {
    counter: { value: 10 } // App start mein counter 10 se shuru hoga
  }
});

export default store;

// store type check karna:
// store.getState() → poora state tree
// store.dispatch() → actions dispatch karna
// store.subscribe() → state changes sunna
```

**configureStore kya automatically karta hai:**

```
configureStore automatically:
1. ✅ redux-thunk middleware add karta hai
2. ✅ Redux DevTools Extension enable karta hai (development mein)
3. ✅ Immutability check middleware add karta hai (dev mein)
4. ✅ Serializability check middleware add karta hai (dev mein)
```

**Immutability Middleware kya karta hai?**
```javascript
// Agar tum yeh karo:
dispatch({ type: "test" });
state.someValue = "changed"; // Direct mutation!

// Immutability middleware WARN karega:
// "A state mutation was detected between dispatches..."
// Yeh development mein bahut helpful hai bugs pakadne ke liye!
```

---

### 💻 4.4 React App Mein Store Connect Karna

```javascript
// src/index.js (React 18) ya src/main.jsx (Vite)
import React from 'react';
import ReactDOM from 'react-dom/client';
import { Provider } from 'react-redux'; // React-Redux ka bridge
import store from './app/store';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));

root.render(
  <React.StrictMode>
    {/* Provider store ko poori app mein available banata hai */}
    {/* Exactly jaise Context.Provider kaam karta hai */}
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
```

**Provider kya karta hai:**
```
Provider = Context ke through store ko pass karta hai poori component tree mein
           Koi bhi component useSelector/useDispatch use kar sakta hai bina props ke
```

---

### 💻 4.5 Redux DevTools Extension

Browser mein install karo:
- Chrome: "Redux DevTools" extension
- Firefox: "Redux DevTools" extension

```javascript
// store.js — DevTools automatically kaam karte hain configureStore se
// Koi extra configuration nahi chahiye!
const store = configureStore({
  reducer: { counter: counterReducer }
  // devTools: true → default in development
});
```

**DevTools se kya dekh sakte ho:**
```
DevTools Features:
├── State tab → Current store state tree
├── Actions tab → Dispatch hua har action list
├── Diff tab → State change kya hua har action pe
├── Jump to state → Kisi bhi past state pe jaao (Time Travel!)
└── Skip actions → Kuch actions ignore karo debugging ke liye
```

---

### ⚠️ 4.6 Common Mistakes

**Mistake 1: Provider ke bahar useSelector use karna**
```javascript
// ❌ Wrong — Provider ke bina
function App() {
  const count = useSelector(state => state.counter.value); // ERROR!
  return <div>{count}</div>;
}

// App ko Provider ke bahar render kiya

// ✅ Correct — Provider wrap karo
ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```

**Mistake 2: Multiple Stores Banana**
```javascript
// ❌ Wrong — Redux mein sirf ek store hona chahiye
const counterStore = configureStore({ reducer: { counter: counterReducer } });
const cartStore = configureStore({ reducer: { cart: cartReducer } });

// ✅ Correct — Ek store mein combine karo
const store = configureStore({
  reducer: {
    counter: counterReducer,
    cart: cartReducer
  }
});
```

---

### 💡 4.7 Pro Tip

```
Production mein DevTools disable karna important hai security ke liye.
configureStore automatically yeh handle karta hai:
- NODE_ENV=development → DevTools ON
- NODE_ENV=production → DevTools OFF

Lekin manually bhi set kar sakte ho:
devTools: process.env.NODE_ENV !== 'production'
```

---

### 🎯 4.8 Interview Questions — Module 4

**Q1: configureStore aur createStore mein kya fark hai?**  
**A:** `createStore` plain Redux ka function hai — manually middleware aur DevTools setup karna padta hai. `configureStore` RTK ka function hai — automatically thunk, DevTools, immutability checks add karta hai. RTK use karte samay hamesha `configureStore` use karo.

**Q2: Provider kya karta hai aur kyun zaroori hai?**  
**A:** `Provider` React-Redux ka component hai jo Redux store ko React component tree mein available banata hai Context API ke through. Bina Provider ke koi bhi component `useSelector` ya `useDispatch` use nahi kar sakta.

**Q3: redux-thunk kya hai?**  
**A:** Redux middleware hai jo functions dispatch karne allow karta hai (sirf plain objects ki jagah). Async operations ke liye use hota hai — API calls, setTimeout, etc. RTK `configureStore` mein automatically add hota hai.

**Q4: DevTools production mein kyun disable hone chahiye?**  
**A:** Security reasons — DevTools se attackers poori app state dekh sakte hain, sensitive data leak ho sakta hai. RTK automatically production mein disable karta hai.

---

### 📋 4.9 Practice Task

1. Create-React-App ya Vite se naya React project banao
2. `@reduxjs/toolkit` aur `react-redux` install karo
3. `store.js` banao `configureStore` se
4. `index.js` mein `Provider` wrap karo
5. Browser mein Redux DevTools install karo aur store observe karo

---

## 📚 MODULE 5: createSlice Deep Dive — Reducers, Action Creators, Immer

---

### 🧠 5.1 Concept Explanation — createSlice Kya Karta Hai

`createSlice` ek powerful RTK function hai jo ek saath:
1. Reducer function generate karta hai
2. Action creators generate karta hai
3. Action types generate karta hai

**"Slice"** = State ka ek portion + us portion ke liye logic

Jaise ek pizza ka slice = pizza ka ek hissa + toppings (logic)

---

### 💻 5.2 createSlice — Poora Syntax Explain

```javascript
// features/counter/counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

// createSlice ko ek configuration object dena padta hai
const counterSlice = createSlice({
  
  // ---- name ----
  // Yeh action types ka prefix banega
  // "counter" name → action types: "counter/increment", "counter/decrement"
  name: 'counter',

  // ---- initialState ----
  // Reducer ki initial state — pehli baar store mein yahi hoga
  initialState: {
    value: 0,
    status: 'idle' // 'idle' | 'loading' | 'failed'
  },

  // ---- reducers ----
  // Yahan ek object dete hain jisme functions hain
  // Har function = ek reducer case
  // Har function ko:
  //   - 1st arg: current state (Immer draft)
  //   - 2nd arg: action object { type, payload }
  reducers: {
    
    // Simple case — sirf state change karo
    increment(state) {
      // Yeh mutation lagti hai lekin Immer ke through safe hai!
      state.value += 1;
    },
    
    decrement(state) {
      state.value -= 1;
    },

    // Payload ke saath
    incrementByAmount(state, action) {
      // action.payload = dispatch ke time jo value doge
      state.value += action.payload;
    },

    // Complex state update
    reset(state) {
      state.value = 0;
      state.status = 'idle';
    },

    // Array operations — Immer se safe hain
    addItem(state, action) {
      // Direct push — Immer internally immutable banata hai
      state.items.push(action.payload);
    },

    removeItem(state, action) {
      // Filter karke naya array banana — dono tarike kaam karte hain RTK mein
      // Option 1: Mutating approach (Immer handles it)
      const index = state.items.findIndex(item => item.id === action.payload);
      if (index !== -1) {
        state.items.splice(index, 1);
      }
    },

    updateItem(state, action) {
      const { id, changes } = action.payload;
      const item = state.items.find(item => item.id === id);
      if (item) {
        // Object directly update karo — Immer handles
        Object.assign(item, changes);
      }
    }
  }
});

// ---- ACTION CREATORS EXTRACT KARNA ----
// createSlice automatically action creators banata hai
// Har reducer function ke liye ek action creator
export const {
  increment,
  decrement,
  incrementByAmount,
  reset,
  addItem,
  removeItem,
  updateItem
} = counterSlice.actions;

// ---- REDUCER EXPORT ----
// Yeh store mein configure karenge
export default counterSlice.reducer;

// Usage examples:
// increment() → { type: "counter/increment" }
// incrementByAmount(5) → { type: "counter/incrementByAmount", payload: 5 }
// addItem({ id: 1, name: "Book" }) → { type: "counter/addItem", payload: { id: 1, name: "Book" } }
```

---

### 🧠 5.3 Immer Ka Role — Yeh Magic Kaise Hota Hai

**Immer** ek library hai jo "mutations" ko internally safe immutable updates mein convert karti hai.

```
Tumhara Code (mutating):          Immer Internally:
state.value += 1;          →      return { ...state, value: state.value + 1 };
state.items.push(item);    →      return { ...state, items: [...state.items, item] };
state.user.name = "Kayum"; →      return { ...state, user: { ...state.user, name: "Kayum" } };
```

**Immer kaise kaam karta hai — Step by Step:**

```
Step 1: Immer ek "draft" banata hai — state ka proxy copy
           ↓
Step 2: Tum draft modify karte ho (yahi tumhara reducer code hai)
           ↓
Step 3: Immer changes track karta hai
           ↓
Step 4: Immer naya immutable state object banata hai changes se
           ↓
Step 5: Original state untouched rehta hai
```

```javascript
// Immer kaise kaam karta hai internally (simplified):
import { produce } from 'immer';

// Yeh RTK ke andar automatically hota hai
function reducer(state, action) {
  // produce() ek draft banata hai
  return produce(state, (draft) => {
    // Yahan tumhara reducer code run hota hai
    if (action.type === "counter/increment") {
      draft.value += 1; // Draft mutate ho raha hai, state nahi
    }
  });
  // produce() automatically naya immutable object return karta hai
}
```

---

### 🆚 5.4 Immer Mutations vs Return — Dono Kaam Karte Hain

```javascript
const slice = createSlice({
  name: 'example',
  initialState: { value: 0, items: [] },
  reducers: {
    
    // ---- APPROACH 1: Immer mutation (recommended for complex updates) ----
    incrementV1(state) {
      state.value += 1; // Mutation — Immer handles
    },

    // ---- APPROACH 2: Return new state (plain Redux style) ----
    incrementV2(state) {
      return { ...state, value: state.value + 1 }; // Explicit return
    },

    // ⚠️ IMPORTANT: Dono ek saath mat karo!
    // ❌ Wrong — return aur mutation mixed
    incrementWrong(state) {
      state.value += 1; // Mutation
      return state; // Yeh Immer ko confuse karta hai!
      // Agar return karo toh Immer mutation ignore kar deta hai
    }
  }
});
```

**Rule:** Ya toh mutate karo (Immer handles), ya explicitly naya object return karo. Dono ek saath nahi.

---

### 💻 5.5 Complete Feature Slice — User Auth Example

```javascript
// features/auth/authSlice.js
import { createSlice } from '@reduxjs/toolkit';

// State ka initial shape — pehle design karo
const initialState = {
  user: null,          // Logged in user ki info
  isLoggedIn: false,   // Login status
  token: null,         // Auth token
  error: null,         // Login error message
};

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    // Login action — user object aur token milega payload mein
    loginSuccess(state, action) {
      const { user, token } = action.payload;
      state.user = user;
      state.token = token;
      state.isLoggedIn = true;
      state.error = null; // Previous error clear karo
    },

    // Logout action — sab reset
    logout(state) {
      state.user = null;
      state.token = null;
      state.isLoggedIn = false;
      state.error = null;
    },

    // Error set karna — login fail hone pe
    setAuthError(state, action) {
      state.error = action.payload; // error message string
      state.isLoggedIn = false;
    },

    // User profile update
    updateUserProfile(state, action) {
      // Existing user object mein changes merge karo
      if (state.user) {
        Object.assign(state.user, action.payload);
      }
    }
  }
});

export const { loginSuccess, logout, setAuthError, updateUserProfile } = authSlice.actions;

// Selectors — state se data nikalne ke liye
// Inhe slice file mein hi define karo — colocated!
export const selectUser = (state) => state.auth.user;
export const selectIsLoggedIn = (state) => state.auth.isLoggedIn;
export const selectAuthError = (state) => state.auth.error;
export const selectToken = (state) => state.auth.token;

export default authSlice.reducer;
```

---

### ⚠️ 5.6 Common Mistakes with createSlice

**Mistake 1: Slice ke bahar state mutate karna**
```javascript
// ❌ Wrong — component mein direct state mutation
const user = useSelector(selectUser);
user.name = "New Name"; // GALAT! State directly mutate kiya

// ✅ Correct — action dispatch karo
dispatch(updateUserProfile({ name: "New Name" }));
```

**Mistake 2: Payload ko galat tarike se bhejana**
```javascript
// ❌ Wrong — object ke andar object not needed usually
dispatch(increment({ amount: 5 }));
// Reducer: action.payload = { amount: 5 }

// ✅ Correct — direct value
dispatch(incrementByAmount(5));
// Reducer: action.payload = 5

// Note: Complex data ke liye object bilkul theek hai
dispatch(addItem({ id: 1, name: "Book", price: 300 }));
// action.payload = { id: 1, name: "Book", price: 300 }
```

**Mistake 3: createSlice se action types directly hardcode karna**
```javascript
// ❌ Wrong — string hardcode karna
dispatch({ type: "counter/increment" });

// ✅ Correct — auto-generated action creator use karo
dispatch(increment());

// Agar type check karna ho:
console.log(increment.type); // "counter/increment"
```

---

### 💡 5.7 Pro Tips

1. **Slice ke saath selectors colocate karo** — State shape change hone pe sirf ek jagah update karna padega
2. **Slice name meaningful rakho** — Action types mein prefix banega: `auth/loginSuccess`
3. **initialState mein sab fields define karo** — Undefined fields se bugs hote hain
4. **Immer ke limitations jaano** — Non-serializable data (Date, Map, Set, Class instances) ko plain objects/arrays mein convert karo

---

### 🎯 5.8 Interview Questions — Module 5

**Q1: createSlice kya auto-generate karta hai?**  
**A:** Teen cheezein: (1) Reducer function — switch-case ke bina, (2) Action creators — har reducer ke liye ek, (3) Action types — `sliceName/reducerName` format mein.

**Q2: Immer kya hai aur kyun RTK mein use hota hai?**  
**A:** Immer ek library hai jo mutations ko immutable updates mein convert karta hai internally. RTK mein `createSlice` ke andar Immer use hota hai taki developers direct state mutations likh sakein (easy syntax) lekin immutability maintain rahe.

**Q3: Kya createSlice mein return karna zaroori hai?**  
**A:** Nahi. Agar tum state mutate karte ho (Immer ke through), return optional hai. Lekin agar return karte ho toh naya object return karo, mutated state nahi.

**Q4: Slice name kyun important hai?**  
**A:** Slice name action types ka prefix banega. `name: 'counter'` + reducer `increment` → action type `counter/increment`. Agar name conflict kare, action types duplicate ho sakte hain.

**Q5: Action payload kaise customize karein?**  
**A:** `prepare` callback use karo — advanced use case. Default mein pehla argument `payload` ban jaata hai. Complex payload ke liye `prepare` callback mein custom logic likh sakte ho.

---

### 📋 5.9 Practice Task

Ek `notificationsSlice` banao:
- `initialState`: `{ items: [], unreadCount: 0 }`
- Actions:
  - `addNotification(state, action)` — notification push karo, unreadCount++
  - `markAsRead(state, action)` — id se notification find karo, `read: true` karo, unreadCount--
  - `removeNotification(state, action)` — id se remove karo
  - `clearAll(state)` — sab clear karo, unreadCount = 0
- Selectors export karo: `selectAllNotifications`, `selectUnreadCount`

---

## 📚 MODULE 6: React App Mein Redux Toolkit Integrate Karna

---

### 🧠 6.1 Concept Explanation — React aur Redux Ko Milana

RTK store banana ek baat hai, React components mein use karna doosri. `react-redux` library yeh bridge provide karta hai:

1. **`useSelector`** — Store se data read karna
2. **`useDispatch`** — Actions dispatch karna

---

### 🌍 6.2 Real-World Analogy

Redux Store = Ek bank account  
`useSelector` = ATM se balance check karna  
`useDispatch` = ATM se transaction karna

---

### 💻 6.3 useSelector — Store Se Data Padhna

```javascript
// ---- useSelector kya hai ----
// useSelector(selectorFunction) hook hai jo:
// 1. Store se current state nikalta hai
// 2. Component ko woh data deta hai
// 3. Jab woh data change ho, component re-render karta hai

import { useSelector } from 'react-redux';

// Component mein use karna:
function CounterDisplay() {
  // State se counter.value nikalo
  // State → "counter" slice → "value" field
  const count = useSelector((state) => state.counter.value);
  
  return <h1>Count: {count}</h1>;
}

// ---- Selector function alag define karna (better practice) ----
// counterSlice.js mein:
export const selectCount = (state) => state.counter.value;

// Component mein:
import { selectCount } from './features/counter/counterSlice';

function CounterDisplay() {
  const count = useSelector(selectCount); // Cleaner!
  return <h1>Count: {count}</h1>;
}
```

**useSelector ka internal mechanism:**
```
Pehla render → selector run hota hai → value milti hai → component render

Store update hone pe:
→ useSelector apna selector dobara run karta hai
→ Agar naya value !== purana value (strict equality)
→ Component re-render hota hai
→ Agar same value → No re-render (optimized!)
```

---

### 💻 6.4 useDispatch — Actions Bhejna

```javascript
import { useDispatch } from 'react-redux';
import { increment, decrement, incrementByAmount } from './features/counter/counterSlice';

function CounterControls() {
  // dispatch function milta hai
  const dispatch = useDispatch();
  
  // Event handlers mein dispatch karo
  const handleIncrement = () => {
    dispatch(increment()); // Action creator call karo
  };

  const handleDecrement = () => {
    dispatch(decrement());
  };

  const handleIncrementBy10 = () => {
    dispatch(incrementByAmount(10)); // Payload ke saath
  };

  return (
    <div>
      <button onClick={handleDecrement}>-</button>
      <button onClick={handleIncrement}>+</button>
      <button onClick={handleIncrementBy10}>+10</button>
    </div>
  );
}
```

---

### 💻 6.5 Complete Working App — Counter with Redux

**Folder Structure:**
```
src/
├── app/
│   └── store.js           ← Redux store
├── features/
│   └── counter/
│       ├── counterSlice.js  ← Slice (reducer + actions + selectors)
│       └── Counter.jsx      ← Feature component
├── App.jsx
└── main.jsx (Vite) / index.js (CRA)
```

```javascript
// src/app/store.js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from '../features/counter/counterSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});
```

```javascript
// src/features/counter/counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment(state) { state.value += 1; },
    decrement(state) { state.value -= 1; },
    incrementByAmount(state, action) { state.value += action.payload; },
    reset(state) { state.value = 0; },
  },
});

export const { increment, decrement, incrementByAmount, reset } = counterSlice.actions;

// Selectors
export const selectCount = (state) => state.counter.value;

export default counterSlice.reducer;
```

```javascript
// src/features/counter/Counter.jsx
import React, { useState } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import {
  increment,
  decrement,
  incrementByAmount,
  reset,
  selectCount
} from './counterSlice';

function Counter() {
  // Local state sirf UI ke liye (input field)
  const [incrementAmount, setIncrementAmount] = useState('2');
  
  // Redux store se data
  const count = useSelector(selectCount);
  
  // Dispatch function
  const dispatch = useDispatch();

  return (
    <div style={{ padding: '20px' }}>
      <h2>Redux Counter</h2>
      
      {/* Count display */}
      <p style={{ fontSize: '2rem', fontWeight: 'bold' }}>
        {count}
      </p>
      
      {/* Main controls */}
      <div>
        <button onClick={() => dispatch(decrement())}>-</button>
        <button onClick={() => dispatch(increment())}>+</button>
        <button onClick={() => dispatch(reset())}>Reset</button>
      </div>
      
      {/* Custom amount */}
      <div style={{ marginTop: '10px' }}>
        <input
          type="number"
          value={incrementAmount}
          onChange={(e) => setIncrementAmount(e.target.value)}
          style={{ width: '60px' }}
        />
        <button
          onClick={() => dispatch(incrementByAmount(Number(incrementAmount)))}
        >
          Add Amount
        </button>
      </div>
    </div>
  );
}

export default Counter;
```

```javascript
// src/App.jsx
import Counter from './features/counter/Counter';

function App() {
  return (
    <div>
      <h1>My Redux App</h1>
      {/* Multiple instances — same store! */}
      <Counter />
    </div>
  );
}

export default App;
```

```javascript
// src/main.jsx (Vite)
import React from 'react';
import ReactDOM from 'react-dom/client';
import { Provider } from 'react-redux';
import { store } from './app/store';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
```

---

### 💻 6.6 Folder Structure Best Practices

**Feature-Based (Recommended):**
```
src/
├── app/
│   └── store.js
├── features/
│   ├── auth/
│   │   ├── authSlice.js      ← State + reducers + selectors
│   │   ├── LoginForm.jsx     ← Component
│   │   ├── authAPI.js        ← API calls (optional)
│   │   └── index.js          ← Public exports
│   ├── cart/
│   │   ├── cartSlice.js
│   │   ├── CartPage.jsx
│   │   └── CartItem.jsx
│   └── products/
│       ├── productsSlice.js
│       ├── ProductList.jsx
│       └── ProductCard.jsx
├── components/               ← Shared/reusable components
│   ├── Button.jsx
│   └── Modal.jsx
└── App.jsx
```

**Kyun feature-based?**
- Har feature apni jagah — easy to find
- Team mein different members different features pe kaam kar sakte hain
- Feature delete karna easy — sirf woh folder hatao

---

### ⚠️ 6.7 Common Mistakes

**Mistake 1: useSelector mein unnecessary subscriptions**
```javascript
// ❌ Wrong — poora state subscribe kar rahe hain
const state = useSelector(state => state); // Koi bhi change → re-render!

// ✅ Correct — sirf zaroori data
const count = useSelector(state => state.counter.value); // Sirf count change pe re-render
```

**Mistake 2: useDispatch ke unnecessary re-creations**
```javascript
// ❌ Wrong — dispatch render mein create mat karo
function Component() {
  // dispatch ek hi baar milni chahiye
  const dispatch = useDispatch(); // Yeh actually correct hai!
  // dispatch reference stable hai — re-render pe same reference milta hai
}
```

**Mistake 3: Redux mein local state dalna**
```javascript
// ❌ Wrong — form input ka local state Redux mein
dispatch(setInputValue(e.target.value)); // Har keystroke pe dispatch!

// ✅ Correct — local UI state = useState
const [value, setValue] = useState('');
// Sirf submit pe dispatch karo
```

---

### 🎯 6.8 Interview Questions — Module 6

**Q1: useSelector aur useDispatch ka kya kaam hai?**  
**A:** `useSelector` store se data read karta hai — ek selector function pass karo jo state se required data extract kare. `useDispatch` dispatch function deta hai jisse actions dispatch karo. Dono `react-redux` ke hooks hain.

**Q2: useSelector kab component ko re-render karta hai?**  
**A:** Jab selector ka return value change ho (strict equality check `===`). Agar same reference ya same primitive value aaye, re-render nahi hota. Isliye specific data select karo, poora state nahi.

**Q3: kya ek component mein multiple useSelector calls kar sakte hain?**  
**A:** Haan, bilkul. Har `useSelector` independently subscribe karta hai apne selected value ke liye.

**Q4: Local state aur Redux state mein kya decide karna chahiye kab?**  
**A:** Rule of thumb: Agar state sirf ek component mein use ho → useState. Agar multiple components share karte hain ya route change pe persist karna ho → Redux.

---

### 📋 6.9 Practice Task

Complete React + RTK app banao — **Todo App:**
1. `todosSlice.js` banao: add, remove, toggle, clear actions
2. `TodoForm.jsx` — input aur add button
3. `TodoList.jsx` — todos render karo `useSelector` se
4. `TodoItem.jsx` — individual todo, toggle aur remove buttons
5. Sab components alag files mein, feature-based folder structure mein

---

## 📚 MODULE 7: extraReducers — Dusre Slices Ke Actions Handle Karna

---

### 🧠 7.1 Concept Explanation — Cross-Slice Communication

`reducers` field mein jo functions likhte ho, woh sirf us slice ke liye hote hain. Lekin kya hoga jab ek slice ko **doosre slice ka action** handle karna ho?

**Example:** User logout karta hai → Cart bhi clear ho jaye, Notifications bhi clear hon.

Iske liye hai `extraReducers`.

---

### 🌍 7.2 Real-World Analogy

Socho "Logout" ek announcement hai:
- HR department sunti hai → Employee record clear karo
- IT department sunti hai → Access revoke karo  
- Accounts department sunti hai → Payroll freeze karo

Ek action, multiple departments (slices) react karti hain.

---

### 💻 7.3 extraReducers — Builder Syntax (Recommended)

```javascript
// features/cart/cartSlice.js
import { createSlice } from '@reduxjs/toolkit';
import { logout } from '../auth/authSlice'; // Doosre slice ka action import

const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: [],
    total: 0,
    coupon: null
  },
  
  // Cart ke apne actions
  reducers: {
    addItem(state, action) {
      state.items.push(action.payload);
      state.total += action.payload.price;
    },
    removeItem(state, action) {
      const index = state.items.findIndex(item => item.id === action.payload);
      if (index !== -1) {
        state.total -= state.items[index].price;
        state.items.splice(index, 1);
      }
    },
  },

  // ---- extraReducers — doosre slices ke actions handle karna ----
  // Builder pattern use karo (modern aur type-safe)
  extraReducers: (builder) => {
    
    // Jab user logout kare, cart bhi clear ho
    builder.addCase(logout, (state) => {
      // logout authSlice ka action creator hai
      state.items = [];
      state.total = 0;
      state.coupon = null;
    });
    
    // Multiple actions handle kar sakte ho
    // builder.addCase(someOtherAction, (state, action) => { ... });
    
    // addMatcher — specific conditions pe handle karna
    // builder.addMatcher(
    //   (action) => action.type.endsWith('/fulfilled'),
    //   (state, action) => { ... }
    // );
    
    // addDefaultCase — koi bhi action jo match na kare
    // builder.addDefaultCase((state, action) => { ... });
  }
});

export const { addItem, removeItem } = cartSlice.actions;
export default cartSlice.reducer;
```

```javascript
// features/notifications/notificationsSlice.js
import { createSlice } from '@reduxjs/toolkit';
import { logout } from '../auth/authSlice';

const notificationsSlice = createSlice({
  name: 'notifications',
  initialState: { items: [], unreadCount: 0 },
  reducers: {
    addNotification(state, action) {
      state.items.unshift(action.payload);
      state.unreadCount++;
    },
  },
  extraReducers: (builder) => {
    // Logout pe notifications bhi clear
    builder.addCase(logout, (state) => {
      state.items = [];
      state.unreadCount = 0;
    });
  }
});

export const { addNotification } = notificationsSlice.actions;
export default notificationsSlice.reducer;
```

```javascript
// features/auth/authSlice.js
import { createSlice } from '@reduxjs/toolkit';

const authSlice = createSlice({
  name: 'auth',
  initialState: { user: null, isLoggedIn: false },
  reducers: {
    loginSuccess(state, action) {
      state.user = action.payload;
      state.isLoggedIn = true;
    },
    logout(state) {
      // Sirf auth state clear karo
      state.user = null;
      state.isLoggedIn = false;
    }
  }
});

export const { loginSuccess, logout } = authSlice.actions;
export default authSlice.reducer;
```

```javascript
// Test karo — ek dispatch se sab clear:
dispatch(logout());
// Auth slice: user null, isLoggedIn false
// Cart slice: items [], total 0
// Notifications slice: items [], unreadCount 0
```

---

### 💻 7.4 extraReducers ke saath createAsyncThunk (Preview)

```javascript
// Yeh Module 8-9 mein detail mein cover hoga
// Abhi sirf preview:

import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Async action banana
export const fetchUsers = createAsyncThunk('users/fetchAll', async () => {
  const response = await fetch('/api/users');
  return response.json();
});

const usersSlice = createSlice({
  name: 'users',
  initialState: { data: [], loading: false, error: null },
  reducers: {},
  
  extraReducers: (builder) => {
    // Async action ke 3 states handle karna
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;  // API call shuru
        state.error = null;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.loading = false;
        state.data = action.payload; // API se data aaya
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message; // Error store karo
      });
  }
});
```

---

### ⚠️ 7.5 Common Mistakes

**Mistake 1: Circular imports**
```javascript
// ❌ Wrong — circular dependency bana sakta hai
// authSlice.js mein cartSlice import karo aur cartSlice.js mein authSlice import karo
// Aisa mat karo!

// ✅ Correct — sirf ek direction mein import karo
// cartSlice → authSlice ka action use karo (not vice versa)
// Ya shared actions file banao
```

**Mistake 2: extraReducers mein slice ke apne actions handle karna**
```javascript
// ❌ Unnecessarily complicated
extraReducers: (builder) => {
  builder.addCase(cartSlice.actions.addItem, (state, action) => {
    // Yeh cartSlice ke andar unnecessary hai — use 'reducers' field!
  });
}

// ✅ Apne actions ke liye 'reducers' field use karo
reducers: {
  addItem(state, action) { ... }
}
```

---

### 🎯 7.6 Interview Questions — Module 7

**Q1: extraReducers kab use karte hain?**  
**A:** Jab ek slice ko doosre slice ke actions par react karna ho. Example: user logout karne pe cart clear karna — cart slice auth slice ke logout action ko extraReducers mein handle karta hai.

**Q2: reducers aur extraReducers mein kya fark hai?**  
**A:** `reducers` field mein jo functions likhte ho woh slice ke "own" actions banate hain — action creators automatically generate hote hain. `extraReducers` mein bahar se aaye actions handle karte hain — koi new action creator nahi banta.

**Q3: Builder pattern aur object notation mein kya fark hai?**  
**A:** Builder pattern modern approach hai — `builder.addCase()`, `builder.addMatcher()`, `builder.addDefaultCase()` methods available hain. Object notation deprecated ho raha hai. Builder pattern use karo.

---

### 📋 7.7 Practice Task

1. `authSlice.js` banao — `login` aur `logout` actions
2. `profileSlice.js` banao — `extraReducers` mein:
   - `login` action pe profile data load karo
   - `logout` action pe profile clear karo
3. `wishlistSlice.js` banao — `extraReducers` mein:
   - `logout` pe wishlist clear karo
4. Verify karo ki ek `dispatch(logout())` se teen slices update hote hain
