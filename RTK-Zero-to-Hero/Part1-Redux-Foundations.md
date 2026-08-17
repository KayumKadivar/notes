# React Redux Toolkit (RTK) & RTK Query: Zero to Hero
### A Complete Interview-Ready Hinglish Guide

> **Language:** JavaScript only (No TypeScript)  
> **Style:** Hinglish — jaise mentor se seekh rahe ho  
> **Level:** Beginner → Production-Ready

---

# 📚 PART 1: REDUX FOUNDATIONS

---

## 📚 MODULE 1: Redux Kya Hai Aur Kyun Chahiye

---

### 🧠 1.1 Concept Explanation — State Management Ki Problem

#### State Management Problem Kya Hai?

Socho tumhara ek React app hai jisme 50 components hain. Jab user login karta hai, toh uska naam, email, aur role — yeh sab information multiple components ko chahiye hoti hai:

- `Header` component — username dikhane ke liye
- `Sidebar` component — menu items control karne ke liye  
- `Dashboard` component — personalized content ke liye
- `ProfilePage` component — user details ke liye

Ab yeh problem hai: **yeh data kahan rakho aur kaise share karo?**

#### Prop Drilling — Sabse Pehli Aur Buri Problem

```
App (user data yahan hai)
  └── Layout
        └── Sidebar
              └── UserMenu
                    └── Avatar (yahan chahiye user ka naam!)
```

Is structure mein agar `Avatar` ko user ka naam chahiye, toh:
- `App` → `Layout` → `Sidebar` → `UserMenu` → `Avatar`

Har intermediate component ko props pass karne padte hain, **chahe unhe khud use nahi karna**. Yeh hai **Prop Drilling** — ek bada problem.

```javascript
// Yeh kaisa dikhta hai — bahut ugly aur maintain karna mushkil
function App() {
  const user = { name: "Kayum", role: "admin" };
  return <Layout user={user} />;
}

function Layout({ user }) {
  // Layout ko khud user ki zaroorat nahi, bas pass kar raha hai
  return <Sidebar user={user} />;
}

function Sidebar({ user }) {
  // Sidebar ko bhi khud nahi chahiye
  return <UserMenu user={user} />;
}

function UserMenu({ user }) {
  // Yeh bhi sirf pass kar raha hai
  return <Avatar user={user} />;
}

function Avatar({ user }) {
  // Aakhir yahan use ho raha hai!
  return <img src={user.avatar} alt={user.name} />;
}
```

**Yeh kyun problem hai?**
1. Har component mein `user` prop add karna padta hai
2. Agar structure change ho, toh har jagah update karna padta hai
3. Components tightly coupled ho jaate hain
4. Reusability kam ho jaati hai

#### Single Source of Truth

Redux ka pehla principle: **ek hi jagah (Store) mein poori app ka state rakho.**

Jaise ek school mein **Principal Office** hoti hai — wahan poori school ki information hoti hai. Koi bhi teacher wahan jaake student ki information le sakta hai. Principal Office ko har class mein jaane ki zaroorat nahi.

Redux mein yeh Principal Office = **Store** hai.

#### Predictable State Container

Redux ka doosra principle: **state sirf ek defined process se change hogi.**

```
State Change Process:
Action Dispatch → Reducer → New State → UI Update
```

Iska matlab: **koi bhi unexpected state change nahi hogi.** Tum always trace kar sakte ho ki state kab, kyon aur kaise change hua.

---

### 🌍 1.2 Real-World Example — Library Management System

Socho ek library hai:

**Bina Redux ke (chaos):**
- Koi bhi book directly shelf se utha sakta hai
- Return karne ka koi record nahi
- Pata nahi kaunsi book kahan hai
- Ek hi book multiple log le jaate hain

**Redux ke saath (organized):**
- Library ka ek **central database** (Store)
- Book leni hai toh **request form** bharo (Action)  
- Librarian form check karke **record update** karta hai (Reducer)
- Sab transparent aur trackable hai

---

### 🔍 1.3 Redux Kab Use Karna Chahiye?

Redux **zaroorat** kab padti hai:

| Situation | Redux Chahiye? |
|-----------|---------------|
| Simple to-do app | ❌ Nahi |
| Single page, 2-3 components | ❌ Nahi |
| Multiple pages, shared state | ✅ Haan |
| Real-time data (notifications, cart) | ✅ Haan |
| Complex user roles aur permissions | ✅ Haan |
| Large team, multiple features | ✅ Haan |

---

### ⚠️ 1.4 Common Misconceptions

**Galat Soch:** "Redux sirf large apps ke liye hai"  
**Sahi Baat:** Agar aapki app mein shared state hai jo multiple components use karti hain, Redux helpful hai.

**Galat Soch:** "Redux bahut complex hai"  
**Sahi Baat:** Plain Redux complex tha. RTK (Redux Toolkit) ne isko simple bana diya.

---

### 🎯 1.5 Interview Questions — Module 1

**Q1: State management kya hai aur kyun zaroori hai?**  
**A:** State management matlab app ka data organize karna aur manage karna. Jab multiple components ek hi data share karte hain, toh centralized state management chahiye hoti hai taki data consistent rahe aur prop drilling avoid ho.

**Q2: Prop drilling kya hai? Iska solution kya hai?**  
**A:** Prop drilling tab hota hai jab data parent se bahut saare intermediate components ke through pass karna padta hai, chahe woh components data use na karein. Solution: Redux/Context API use karo jo direct access deta hai kisi bhi component ko.

**Q3: Single Source of Truth ka kya matlab hai?**  
**A:** Poori application ka state ek hi jagah (Redux Store) mein rakha jaata hai. Koi bhi component directly state modify nahi karta — sirf defined actions ke through.

**Q4: Redux kab use karna chahiye aur kab nahi?**  
**A:** Use karo: large apps, shared state, complex data flow. Mat use karo: simple apps, local state jo sirf ek component use kare, jahan Context API kafi ho.

**Q5: Redux ke alternatives kya hain?**  
**A:** Context API (built-in React), Zustand, Jotai, MobX, Recoil. Redux RTK best hai jab: large-scale app ho, time-travel debugging chahiye, team badi ho.

---

### 📋 1.6 Practice Task

Ek simple React app banao (bina Redux ke) jisme:
1. User login karta hai (naam aur role)
2. Yeh information 4-5 nested components mein dikhani hai
3. Prop drilling ka pain mahsoos karo

Phir next module padho aur socho: "Redux se yeh kaise aasan hoga?"

---

## 📚 MODULE 2: Redux Ke Core Concepts

---

### 🧠 2.1 Concept Explanation — Redux Ki Bhasha

Redux samajhne ke liye 5 core concepts samajhne padte hain:

1. **Store** — State ka ghar
2. **Action** — Kya hua? (event)
3. **Reducer** — Kaise change hoga state?
4. **Dispatch** — Action bhejne ka tarika
5. **Selector** — State se data nikalne ka tarika

#### 2.1.1 Store — State Ka Ghar

Store = ek JavaScript object jisme poori app ka state hai.

```javascript
// Store kuch aisa dikhta hai andar se
{
  user: { name: "Kayum", role: "admin", isLoggedIn: true },
  cart: { items: [], total: 450 },
  notifications: { count: 3, messages: [] }
}
```

Store ek single object hai — yahi "Single Source of Truth" hai.

#### 2.1.2 Action — Kya Hua?

Action ek plain JavaScript object hai jo batata hai: **"app mein kya hua"**.

```javascript
// Action ka structure hamesha aisa hota hai:
{
  type: "SOMETHING_HAPPENED",  // zaroori field
  payload: { ... }             // optional: extra data
}
```

Real examples:
```javascript
// User ne login kiya
{ type: "user/loggedIn", payload: { name: "Kayum", role: "admin" } }

// Cart mein item add hua
{ type: "cart/itemAdded", payload: { id: 1, name: "Laptop", price: 50000 } }

// Notification dismiss ki
{ type: "notifications/dismissed", payload: { id: "notif-123" } }
```

**Action sirf describe karta hai kya hua — actual state change nahi karta.**

#### 2.1.3 Reducer — State Kaise Change Hoga

Reducer ek **pure function** hai jo:
- Current state leta hai
- Action leta hai  
- **Naya state return karta hai** (original ko modify nahi karta)

```javascript
// Reducer ka basic structure
function counterReducer(state = { value: 0 }, action) {
  switch (action.type) {
    case "counter/incremented":
      // Naya object return karte hain, purana modify nahi karte
      return { value: state.value + 1 };
    
    case "counter/decremented":
      return { value: state.value - 1 };
    
    case "counter/incrementedByAmount":
      return { value: state.value + action.payload };
    
    default:
      // Har reducer mein default case zaroori hai
      return state;
  }
}
```

#### 2.1.4 Pure Functions — Redux Ka Foundation

Pure function ka matlab:
1. Same input → Always same output
2. No side effects (no API calls, no localStorage, no random values)

```javascript
// ✅ Pure Function — Redux mein yahi use karna hai
function add(a, b) {
  return a + b; // Same input, always same output
}

// ❌ Impure Function — Redux mein allowed nahi
function addWithRandom(a) {
  return a + Math.random(); // Har baar alag result!
}

// ❌ Impure Function — Side effect hai (state modify)
function addToArray(arr, item) {
  arr.push(item); // Original array modify ho gaya!
  return arr;
}

// ✅ Pure version
function addToArray(arr, item) {
  return [...arr, item]; // Naya array, original safe hai
}
```

**Kyun pure functions?** Kyunki Redux mein state predictable honi chahiye. Agar reducer impure ho, toh same action ek baar alag result de sakta hai — yeh debugging ka nightmare hai.

#### 2.1.5 Immutability — State Ko Seedha Mat Chherao

Immutability matlab: **existing state ko kabhi directly modify mat karo, hamesha naya object banao.**

```javascript
// ❌ Wrong — state mutate kar rahe hain
function badReducer(state = { count: 0 }, action) {
  if (action.type === "increment") {
    state.count += 1; // GALAT! state ko directly modify kiya
    return state;
  }
  return state;
}

// ✅ Correct — naya state object return kar rahe hain
function goodReducer(state = { count: 0 }, action) {
  if (action.type === "increment") {
    return { ...state, count: state.count + 1 }; // Naya object!
  }
  return state;
}
```

**Kyun immutability important hai?**

```
State Change Flow:
Old State → Reducer → New State

React agar purana object aur naya object compare kare:
- Same reference → "kuch change nahi hua" → Re-render nahi
- Different reference → "kuch change hua" → Re-render!

Agar hum state mutate karein, reference same rahegi → React ko pata nahi chalega change hua!
```

#### 2.1.6 Dispatch — Action Bhejne Ka Tarika

Dispatch ek function hai jo action ko store tak pahunchata hai.

```javascript
// Action dispatch karna
store.dispatch({ type: "counter/incremented" });
store.dispatch({ type: "cart/itemAdded", payload: { id: 1, name: "Book" } });
```

**Flow:**
```
dispatch(action) → Store receives action → Reducer processes it → New State → UI Updates
```

#### 2.1.7 Selector — State Se Data Nikalna

Selector ek function hai jo store se specific data nikalta hai:

```javascript
// Simple selector
const selectUser = (state) => state.user;
const selectCartTotal = (state) => state.cart.total;

// Computed selector (derived data)
const selectCartItemCount = (state) => state.cart.items.length;
```

---

### 🌍 2.2 Real-World Flow Example — E-Commerce Cart

**Scenario:** User "Add to Cart" button dabata hai.

```
Step 1: User clicks "Add to Cart"
           ↓
Step 2: dispatch({ type: "cart/itemAdded", payload: { id: 5, name: "Phone", price: 20000 } })
           ↓
Step 3: Redux Store receives the action
           ↓
Step 4: cartReducer runs:
        - Current state: { items: [], total: 0 }
        - Action: { type: "cart/itemAdded", payload: { id: 5, ... } }
        - Returns: { items: [{ id: 5, name: "Phone", price: 20000 }], total: 20000 }
           ↓
Step 5: Store updates with new state
           ↓
Step 6: All components using cart state re-render
        - CartIcon shows count: 1
        - CartPage shows item
        - Header shows "1 item"
```

---

### 💻 2.3 Plain Redux — Complete Working Example

```javascript
// ============================================================
// Plain Redux — Bina Redux Toolkit ke (yeh samajhna zaroori hai)
// ============================================================

// npm install redux
const { createStore, combineReducers } = require('redux');

// ---- ACTION TYPES ----
// String constants use karte hain typos avoid karne ke liye
const INCREMENT = 'counter/incremented';
const DECREMENT = 'counter/decremented';
const ADD_TODO = 'todos/added';
const TOGGLE_TODO = 'todos/toggled';

// ---- ACTION CREATORS ----
// Functions jo actions banate hain — best practice hai
function increment() {
  return { type: INCREMENT }; // Action object return karta hai
}

function decrement() {
  return { type: DECREMENT };
}

function addTodo(text) {
  return {
    type: ADD_TODO,
    payload: { id: Date.now(), text, completed: false }
    // Date.now() ek unique id deta hai — real app mein UUID use karo
  };
}

function toggleTodo(id) {
  return { type: TOGGLE_TODO, payload: { id } };
}

// ---- REDUCERS ----

// Counter Reducer
// State parameter mein default value dete hain
function counterReducer(state = { value: 0 }, action) {
  switch (action.type) {
    case INCREMENT:
      // Spread operator se naya object banate hain
      return { ...state, value: state.value + 1 };
    
    case DECREMENT:
      return { ...state, value: state.value - 1 };
    
    default:
      // Default case ZAROORI hai
      return state;
  }
}

// Todos Reducer
function todosReducer(state = { items: [] }, action) {
  switch (action.type) {
    case ADD_TODO:
      return {
        ...state,
        // items array ko mutate karne ki jagah naya array banate hain
        items: [...state.items, action.payload]
      };
    
    case TOGGLE_TODO:
      return {
        ...state,
        items: state.items.map(todo => {
          // Sirf woh todo update karo jiska id match karta hai
          if (todo.id === action.payload.id) {
            return { ...todo, completed: !todo.completed };
          }
          return todo; // Baaki todos as-is return karo
        })
      };
    
    default:
      return state;
  }
}

// ---- COMBINE REDUCERS ----
// Multiple reducers ko ek mein combine karna
const rootReducer = combineReducers({
  counter: counterReducer,  // state.counter
  todos: todosReducer        // state.todos
});

// ---- CREATE STORE ----
const store = createStore(rootReducer);

// ---- SUBSCRIBE ----
// Jab bhi state change ho, yeh function call hoga
store.subscribe(() => {
  console.log('State changed:', store.getState());
});

// ---- DISPATCH ACTIONS ----
store.dispatch(increment());
// State: { counter: { value: 1 }, todos: { items: [] } }

store.dispatch(increment());
// State: { counter: { value: 2 }, todos: { items: [] } }

store.dispatch(addTodo("Redux seekhna"));
// items mein pehla todo add ho gaya

store.dispatch(addTodo("React banao"));
// items mein doosra todo

console.log(store.getState());
// {
//   counter: { value: 2 },
//   todos: { items: [ { id: ..., text: "Redux seekhna", completed: false }, {...} ] }
// }
```

**Line-by-line explanation:**

- `createStore(rootReducer)` — Store banao, rootReducer pass karo
- `combineReducers({...})` — Multiple reducers ko ek root reducer mein merge karo
- `store.subscribe(fn)` — State change hone pe callback call hogi
- `store.dispatch(action)` — Action store ko bhejo
- `store.getState()` — Current state nikalo

---

### 🆚 2.4 Redux vs Context API

| Feature | Redux | Context API |
|---------|-------|-------------|
| Boilerplate | Zyada (plain), Kam (RTK) | Bahut Kam |
| Performance | Optimized (selective re-render) | Sab re-render hote hain |
| DevTools | Excellent (time-travel debug) | Limited |
| Async support | Middleware (Thunk/Saga) | Manual (useEffect) |
| Best for | Large apps, complex state | Small apps, simple sharing |
| Learning curve | Moderate | Low |
| Scalability | High | Medium |

---

### ⚠️ 2.5 Common Mistakes

**Mistake 1: State Mutate Karna**
```javascript
// ❌ Wrong
case "addItem":
  state.items.push(action.payload); // Mutation!
  return state;

// ✅ Correct
case "addItem":
  return { ...state, items: [...state.items, action.payload] };
```

**Mistake 2: Default Case Bhool Jana**
```javascript
// ❌ Wrong — undefined return hoga for unknown actions
function reducer(state, action) {
  switch(action.type) {
    case "increment": return { count: state.count + 1 };
    // default missing!
  }
}

// ✅ Correct
function reducer(state = { count: 0 }, action) {
  switch(action.type) {
    case "increment": return { count: state.count + 1 };
    default: return state; // Zaroori!
  }
}
```

**Mistake 3: Async Code Reducer Mein Likhna**
```javascript
// ❌ Wrong — reducers pure hone chahiye, no side effects
function reducer(state, action) {
  switch(action.type) {
    case "fetchUser":
      fetch('/api/user').then(...); // GALAT! Side effect!
      return state;
  }
}
// ✅ Correct — async ko middleware mein handle karo (thunk/RTK Query)
```

---

### 💡 2.6 Pro Tips

1. **Action types as constants:** Typos avoid karne ke liye always constants use karo
2. **Action creators use karo:** Direct object likhne ki jagah functions banao
3. **State shape design karo pehle:** Store kaise dikhega yeh pehle socho
4. **Reducers ko small rakho:** Har reducer ek feature ke liye

---

### 🎯 2.7 Interview Questions — Module 2

**Q1: Reducer pure function kyun hona chahiye?**  
**A:** Kyunki Redux predictable state changes chahta hai. Pure function: same input → same output, no side effects. Agar reducer impure ho, toh same action different results de sakta hai, jo debugging ko impossible bana deta hai.

**Q2: Immutability Redux mein kyun important hai?**  
**A:** React shallow comparison karta hai state check karne ke liye. Agar state mutate karein, reference same rahegi, React ko pata nahi chalega change hua, aur re-render nahi hoga. Naya object banane se reference change hota hai.

**Q3: Action aur Action Creator mein kya fark hai?**  
**A:** Action ek plain object hai `{ type: "...", payload: ... }`. Action Creator ek function hai jo action object return karta hai. Action creators reusability aur consistency provide karte hain.

**Q4: combineReducers kya karta hai?**  
**A:** Multiple reducers ko ek root reducer mein merge karta hai. Har reducer apni slice of state manage karta hai. State automatically split hoti hai based on keys.

**Q5: Store mein directly state kyon nahi change karte?**  
**A:** Redux ka pattern enforce karta hai ki state sirf dispatch → reducer → new state ke through change ho. Direct mutation se DevTools kaam nahi karte, time-travel debugging nahi hoti, aur React re-renders nahi hote properly.

**Q6: Selector kya hota hai?**  
**A:** Selector ek function hai jo state se specific data extract karta hai. Reusability aur memoization ke liye use hota hai. `const selectUser = (state) => state.user`

---

### 📋 2.8 Practice Task

1. Plain Redux se ek shopping cart banao:
   - `ADD_ITEM` action — item add karo cart mein
   - `REMOVE_ITEM` action — item remove karo
   - `UPDATE_QUANTITY` action — quantity update karo
   - `CLEAR_CART` action — cart saaf karo
2. `combineReducers` use karo: `cart` aur `user` reducers alag-alag
3. `store.subscribe()` se har state change console mein print karo

---

## 📚 MODULE 3: Plain Redux Ki Problems Aur RTK Ka Introduction

---

### 🧠 3.1 Concept Explanation — Plain Redux Ki Takleefein

Plain Redux kaam karta hai, lekin bahut **boilerplate code** likhna padta hai. Ek simple feature ke liye:

1. Action types constants define karo
2. Action creators likhao
3. Reducer likhao (switch-case)
4. Immutable updates likhao (spread operators)
5. Store configure karo
6. Middleware add karo

Yeh sab ek feature ke liye bahut zyada hai!

---

### 🌍 3.2 Real-World Pain — Ek Counter Feature Mein Kitna Code

```javascript
// Plain Redux mein ek simple counter ke liye yeh sab likhna padta tha:

// FILE 1: constants.js
const COUNTER_INCREMENT = 'counter/INCREMENT';
const COUNTER_DECREMENT = 'counter/DECREMENT';
const COUNTER_RESET = 'counter/RESET';
const COUNTER_INCREMENT_BY = 'counter/INCREMENT_BY';

// FILE 2: counterActions.js
export const increment = () => ({ type: COUNTER_INCREMENT });
export const decrement = () => ({ type: COUNTER_DECREMENT });
export const reset = () => ({ type: COUNTER_RESET });
export const incrementBy = (amount) => ({
  type: COUNTER_INCREMENT_BY,
  payload: amount
});

// FILE 3: counterReducer.js
const initialState = { value: 0 };

export function counterReducer(state = initialState, action) {
  switch (action.type) {
    case COUNTER_INCREMENT:
      return { ...state, value: state.value + 1 };
    case COUNTER_DECREMENT:
      return { ...state, value: state.value - 1 };
    case COUNTER_RESET:
      return { ...state, value: 0 };
    case COUNTER_INCREMENT_BY:
      return { ...state, value: state.value + action.payload };
    default:
      return state;
  }
}

// FILE 4: store.js
import { createStore, combineReducers, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import { composeWithDevTools } from 'redux-devtools-extension';

const rootReducer = combineReducers({
  counter: counterReducer,
});

const store = createStore(
  rootReducer,
  composeWithDevTools(applyMiddleware(thunk))
);

// TOTAL: 4 files, ~50 lines sirf ek counter ke liye!
```

### Plain Redux Ki Problems List:

1. **Boilerplate:** Ek feature ke liye bahut zyada code
2. **Action type strings:** Typos ka risk (human error)
3. **Manual immutability:** Nested state update karna bahut mushkil
4. **Middleware setup:** `redux-thunk`, `redux-devtools-extension` manually install aur configure
5. **No async support built-in:** Separately sab karna padta tha
6. **Mutation ka risk:** Easily galti se state mutate ho sakta hai

---

### 💡 3.3 Redux Toolkit — Solution to All Problems

Redux Toolkit (RTK) officially recommended solution hai Redux team ki taraf se. Yahi "Modern Redux" hai.

```
Plain Redux Problems → RTK Solutions:

❌ Boilerplate → ✅ createSlice (ek function mein sab)
❌ Action types manually → ✅ Auto-generated from slice name
❌ Action creators manually → ✅ Auto-generated
❌ Manual immutability → ✅ Immer library built-in (safe mutations)
❌ Middleware setup → ✅ configureStore (auto Thunk + DevTools)
❌ Mutation risk → ✅ Immer catches mutations
```

### RTK mein Counter — Same Feature, Kam Code:

```javascript
// RTK mein POORA counter feature ek hi file mein!
import { createSlice, configureStore } from '@reduxjs/toolkit';

// createSlice ek hi function mein sab kuch generate karta hai
const counterSlice = createSlice({
  name: 'counter',           // slice ka naam — action types mein prefix banega
  initialState: { value: 0 },
  reducers: {
    // Yahan "mutating" code likh sakte hain — Immer internally safe rakhta hai
    increment(state) {
      state.value += 1;
    },
    decrement(state) {
      state.value -= 1;
    },
    reset(state) {
      state.value = 0;
    },
    incrementBy(state, action) {
      state.value += action.payload;
    }
  }
});

// Auto-generated action creators — automatically ban jaate hain!
export const { increment, decrement, reset, incrementBy } = counterSlice.actions;
// increment() → { type: "counter/increment" }
// incrementBy(5) → { type: "counter/incrementBy", payload: 5 }

// Store — ek line mein (thunk + devtools automatically configure hote hain)
const store = configureStore({
  reducer: { counter: counterSlice.reducer }
});

// TOTAL: ~25 lines, ek file! Same functionality!
```

**Yeh magic kaise hota hai?** — Immer library ke through. Agle module mein detail mein padhenge.

---

### 🎯 3.4 Interview Questions — Module 3

**Q1: Plain Redux aur Redux Toolkit mein kya fark hai?**  
**A:** Plain Redux mein bahut saara boilerplate code likhna padta hai — action types, action creators, reducers alag-alag, manual middleware setup. RTK ne sab simplify kiya: `createSlice` se sab auto-generate hota hai, `configureStore` mein thunk aur devtools built-in, Immer se safe mutations.

**Q2: RTK kyun banaya gaya?**  
**A:** Redux team ne khud observe kiya ki developers bahut zyada repetitive code likh rahe the, mutations hoti thi, setup complicated tha. RTK in sab problems ko solve karta hai officially.

**Q3: Immer kya hai aur RTK mein kaise kaam karta hai?**  
**A:** Immer ek library hai jo "draft state" create karta hai. Jab tum state directly mutate karte ho, Immer internally new immutable state banata hai. RTK mein yeh `createSlice` ke andar automatically use hota hai.

---

### 📋 3.5 Practice Task

1. Module 2 ka shopping cart plain Redux mein tha — usse RTK mein convert karo
2. `createSlice` use karo
3. Compare karo: kitni lines Plain Redux mein thi vs RTK mein
4. DevTools install karo browser mein aur state changes observe karo
