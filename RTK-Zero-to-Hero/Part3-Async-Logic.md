# React Redux Toolkit (RTK) & RTK Query: Zero to Hero
## Part 3: Async Logic — createAsyncThunk

---

# 📚 PART 3: ASYNC LOGIC

---

## 📚 MODULE 8: createAsyncThunk — Async Operations In Depth

---

### 🧠 8.1 Concept Explanation — Async Redux Problem

Redux reducer pure function hai — no side effects. Toh API call kaise karein?

**The problem:**
```javascript
// ❌ Yeh kaam nahi karega!
reducers: {
  fetchUsers(state) {
    // Reducer mein async code nahi likh sakte
    const users = await fetch('/api/users'); // GALAT!
    state.users = users;
  }
}
```

**Solution: Middleware**

Redux middleware dispatch ke beech mein intercept karta hai. `redux-thunk` (jo RTK mein built-in hai) ek specific middleware hai jo:
- Normally: sirf plain objects dispatch kar sakte ho
- Thunk ke saath: **functions** bhi dispatch kar sakte ho

```
Normal dispatch:     dispatch({ type: "action" })    → Store
Thunk dispatch:      dispatch(asyncFunction)         → Middleware executes function → Store
```

---

### 🌍 8.2 Real-World Analogy — Restaurant Order

**Bina thunk:** Customer seedha kitchen mein jaake khaana maangta hai. Kitchen direct order nahi le sakti (reducer async nahi le sakta).

**Thunk ke saath:** Waiter (thunk) beech mein kaam karta hai:
1. Customer order deta hai (dispatch)
2. Waiter kitchen ko batata hai (API call)
3. Khaana ready hota hai (response)
4. Waiter table pe serve karta hai (store update)

---

### 🧠 8.3 Thunk Kya Hota Hai — Manual Thunk Example

Pehle samjho manually thunk kaise likhte the:

```javascript
// Manual thunk — ek function jo function return karta hai
function fetchUserThunk(userId) {
  // Inner function thunk hai — dispatch aur getState available hain
  return async function(dispatch, getState) {
    dispatch({ type: 'users/fetchPending' }); // Loading start
    
    try {
      const response = await fetch(`/api/users/${userId}`);
      const user = await response.json();
      dispatch({ type: 'users/fetchFulfilled', payload: user }); // Success
    } catch (error) {
      dispatch({ type: 'users/fetchRejected', payload: error.message }); // Error
    }
  };
}

// Use karna:
store.dispatch(fetchUserThunk(1)); // Function dispatch kar rahe hain!
```

**Yeh boilerplate hai.** `createAsyncThunk` yeh sab handle karta hai.

---

### 💻 8.4 createAsyncThunk — Poora Syntax

```javascript
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';

// ============================================================
// createAsyncThunk(actionTypePrefix, payloadCreator)
// - actionTypePrefix: String — action types ka base name
// - payloadCreator: Async function — actual async work karta hai
// ============================================================

export const fetchUsers = createAsyncThunk(
  // ---- First argument: action type prefix ----
  // Yeh 3 action types generate karta hai:
  // "users/fetchAll/pending"
  // "users/fetchAll/fulfilled"
  // "users/fetchAll/rejected"
  'users/fetchAll',
  
  // ---- Second argument: payloadCreator function ----
  // arg: dispatch ke time jo value pass ki
  // thunkAPI: dispatch, getState, rejectWithValue, etc.
  async (arg, thunkAPI) => {
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/users');
      
      if (!response.ok) {
        // rejectWithValue se custom error message bhejo
        return thunkAPI.rejectWithValue('Failed to fetch users');
      }
      
      const data = await response.json();
      return data; // Yeh action.payload banega fulfilled mein
    } catch (error) {
      return thunkAPI.rejectWithValue(error.message);
    }
  }
);

// fetchUsers ke 3 auto-generated actions:
// fetchUsers.pending  → { type: "users/fetchAll/pending" }
// fetchUsers.fulfilled → { type: "users/fetchAll/fulfilled", payload: data }
// fetchUsers.rejected → { type: "users/fetchAll/rejected", error: {...} }
```

---

### 💻 8.5 Pending/Fulfilled/Rejected — Loading States Handle Karna

```javascript
// features/users/usersSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// ---- Async Thunk Define Karo ----
export const fetchUsers = createAsyncThunk(
  'users/fetchAll',
  async (_, thunkAPI) => {
    // _ = argument nahi chahiye
    const response = await fetch('https://jsonplaceholder.typicode.com/users');
    if (!response.ok) {
      return thunkAPI.rejectWithValue('Server error!');
    }
    return response.json(); // Resolved value = payload
  }
);

export const deleteUser = createAsyncThunk(
  'users/delete',
  async (userId, thunkAPI) => {
    const response = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`, {
      method: 'DELETE'
    });
    if (!response.ok) {
      return thunkAPI.rejectWithValue('Delete failed');
    }
    return userId; // Delete ho gaya userId return karo
  }
);

// ---- Slice ----
const usersSlice = createSlice({
  name: 'users',
  initialState: {
    users: [],
    loading: false,   // Boolean loading state
    error: null,      // Error message
    status: 'idle'    // 'idle' | 'loading' | 'succeeded' | 'failed'
  },
  reducers: {
    // Synchronous actions agar chahiye
    clearError(state) {
      state.error = null;
    }
  },
  extraReducers: (builder) => {
    // ---- fetchUsers states ----
    builder
      .addCase(fetchUsers.pending, (state) => {
        // API call shuru hua — loading UI dikhao
        state.loading = true;
        state.status = 'loading';
        state.error = null; // Previous error clear karo
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        // API call success — data store karo
        state.loading = false;
        state.status = 'succeeded';
        state.users = action.payload; // API response
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        // API call fail — error dikhao
        state.loading = false;
        state.status = 'failed';
        // rejectWithValue se aaya message:
        state.error = action.payload || action.error.message;
      })
    
    // ---- deleteUser states ----
    .addCase(deleteUser.pending, (state) => {
      state.loading = true;
    })
    .addCase(deleteUser.fulfilled, (state, action) => {
      state.loading = false;
      // Deleted user ko array se nikalo
      state.users = state.users.filter(user => user.id !== action.payload);
    })
    .addCase(deleteUser.rejected, (state, action) => {
      state.loading = false;
      state.error = action.payload;
    });
  }
});

export const { clearError } = usersSlice.actions;

// Selectors
export const selectAllUsers = (state) => state.users.users;
export const selectUsersLoading = (state) => state.users.loading;
export const selectUsersError = (state) => state.users.error;
export const selectUsersStatus = (state) => state.users.status;

export default usersSlice.reducer;
```

---

### 💻 8.6 Component Mein Loading aur Error States

```javascript
// features/users/UsersList.jsx
import React, { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import {
  fetchUsers,
  deleteUser,
  selectAllUsers,
  selectUsersLoading,
  selectUsersError,
  selectUsersStatus
} from './usersSlice';

function UsersList() {
  const dispatch = useDispatch();
  const users = useSelector(selectAllUsers);
  const loading = useSelector(selectUsersLoading);
  const error = useSelector(selectUsersError);
  const status = useSelector(selectUsersStatus);

  // Component mount hone pe data fetch karo
  useEffect(() => {
    // Sirf tab fetch karo jab pehle nahi kiya
    if (status === 'idle') {
      dispatch(fetchUsers());
    }
  }, [status, dispatch]);

  // ---- Loading State ----
  if (loading) {
    return (
      <div>
        <p>Loading users...</p>
        {/* Ya koi skeleton loader */}
      </div>
    );
  }

  // ---- Error State ----
  if (error) {
    return (
      <div style={{ color: 'red' }}>
        <p>Error: {error}</p>
        {/* Retry button */}
        <button onClick={() => dispatch(fetchUsers())}>
          Retry
        </button>
      </div>
    );
  }

  // ---- Success State ----
  return (
    <div>
      <h2>Users ({users.length})</h2>
      <ul>
        {users.map(user => (
          <li key={user.id}>
            <span>{user.name} - {user.email}</span>
            <button
              onClick={() => dispatch(deleteUser(user.id))}
              style={{ marginLeft: '10px', color: 'red' }}
            >
              Delete
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default UsersList;
```

---

### 🔄 8.7 Async Thunk Flow — Step by Step

```
User Action / Component Mount
          ↓
dispatch(fetchUsers())  ← Thunk function dispatch kiya
          ↓
redux-thunk middleware intercept karta hai
          ↓
Automatically dispatch: fetchUsers.pending
  → State: { loading: true, status: 'loading', error: null }
  → UI: Loading spinner dikhao
          ↓
payloadCreator function run hota hai (async)
          ↓
  ┌── Success ──────────────────────────────────┐
  │ dispatch: fetchUsers.fulfilled               │
  │ payload = returned data                      │
  │ State: { loading: false, users: [...] }      │
  │ UI: Data dikhao                              │
  └─────────────────────────────────────────────┘
  ┌── Failure ──────────────────────────────────┐
  │ dispatch: fetchUsers.rejected                │
  │ payload = rejectWithValue() ka value         │
  │ State: { loading: false, error: "..." }      │
  │ UI: Error message dikhao                     │
  └─────────────────────────────────────────────┘
```

---

### 💻 8.8 thunkAPI — Kya Kya Available Hai

```javascript
export const createPost = createAsyncThunk(
  'posts/create',
  async (postData, thunkAPI) => {
    // ---- thunkAPI.dispatch ----
    // Doosre actions dispatch kar sakte ho
    thunkAPI.dispatch(setLoading(true));

    // ---- thunkAPI.getState ----
    // Current state padh sakte ho
    const state = thunkAPI.getState();
    const token = state.auth.token; // Auth token lo

    // ---- thunkAPI.rejectWithValue ----
    // Custom error payload bhejo
    if (!token) {
      return thunkAPI.rejectWithValue('Not authenticated');
    }

    try {
      const response = await fetch('/api/posts', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}` // Token use kiya
        },
        body: JSON.stringify(postData)
      });
      
      if (!response.ok) {
        return thunkAPI.rejectWithValue('Failed to create post');
      }
      
      return response.json();
    } catch (error) {
      return thunkAPI.rejectWithValue(error.message);
    }
    
    // ---- thunkAPI.abort ----
    // Thunk ko cancel kar sakte ho (abort controller ke saath)
    
    // ---- thunkAPI.signal ----
    // AbortController signal — network request cancel karne ke liye
  }
);
```

---

### ⚠️ 8.9 Common Mistakes

**Mistake 1: rejectWithValue na use karna**
```javascript
// ❌ Wrong — plain error throw karna
async (arg, thunkAPI) => {
  const response = await fetch('/api/data');
  if (!response.ok) throw new Error('Failed'); // Error object jayega
  // action.error.message milega, action.payload nahi
}

// ✅ Correct — rejectWithValue use karo
async (arg, thunkAPI) => {
  const response = await fetch('/api/data');
  if (!response.ok) {
    return thunkAPI.rejectWithValue('Failed'); // action.payload milega
  }
}
```

**Mistake 2: Har render pe fetch karna**
```javascript
// ❌ Wrong — status check ke bina fetch
useEffect(() => {
  dispatch(fetchUsers()); // Har baar component mount pe fetch!
}, [dispatch]);

// ✅ Correct — status check karo
useEffect(() => {
  if (status === 'idle') { // Sirf pehli baar
    dispatch(fetchUsers());
  }
}, [status, dispatch]);
```

**Mistake 3: Loading state bilkul ignore karna**
```javascript
// ❌ Wrong — UI kuch nahi dikhata loading mein
if (users.length === 0) return <p>No users</p>; // Loading aur empty dono!

// ✅ Correct — loading aur empty alag handle karo
if (loading) return <Spinner />;
if (error) return <ErrorMessage error={error} />;
if (users.length === 0) return <p>No users found</p>;
return <UserList users={users} />;
```

---

### 💡 8.10 Pro Tips

1. **RTK Query prefer karo:** Agar sirf API data chahiye, RTK Query zyada powerful hai (Module 10-14)
2. **createAsyncThunk use karo jab:** Complex async logic ho jo RTK Query se nahi handle hoti
3. **Normalized state use karo:** `state.entities[id]` format large datasets ke liye efficient hai
4. **AbortController:** Long-running requests cancel karne ke liye `thunkAPI.signal` use karo

---

### 🎯 8.11 Interview Questions — Module 8

**Q1: createAsyncThunk kya hai aur kyun use karte hain?**  
**A:** RTK ka utility function hai jo async operations ke liye thunk banata hai. Teen lifecycle action types automatically generate karta hai: pending, fulfilled, rejected. Yeh boilerplate reduce karta hai aur consistent loading/error handling provide karta hai.

**Q2: Pending, fulfilled, rejected kab dispatch hote hain?**  
**A:** `pending` — payloadCreator start hone se pehle. `fulfilled` — payloadCreator successfully complete ho (return value payload banta hai). `rejected` — payloadCreator throw kare ya rejectWithValue call ho.

**Q3: rejectWithValue kyun use karte hain?**  
**A:** Normal error throw karne pe `action.error` mein error info aati hai lekin payload nahi hota. `rejectWithValue(value)` se custom value `action.payload` mein aati hai — zyada control milta hai error handling mein.

**Q4: thunkAPI.getState() kab use karte hain?**  
**A:** Jab async operation mein current state chahiye — jaise auth token nikalna, ya existing data check karna duplicate fetch avoid karne ke liye.

**Q5: createAsyncThunk vs RTK Query — kab kya use karein?**  
**A:** RTK Query: Standard CRUD API operations, caching chahiye, auto re-fetching chahiye. createAsyncThunk: Complex business logic, non-standard async operations, multiple actions orchestrate karne ho.

---

### 📋 8.12 Practice Task

1. JSONPlaceholder API (`https://jsonplaceholder.typicode.com`) use karo
2. `postsSlice.js` banao:
   - `fetchPosts` thunk — GET /posts
   - `createPost` thunk — POST /posts
   - `deletePost` thunk — DELETE /posts/:id
3. Loading, error, success states properly handle karo
4. `PostsList.jsx` component banao jo:
   - Mount pe posts fetch kare
   - Loading spinner dikhae
   - Error state mein retry button dikhae
   - Post delete karne ki functionality ho

---

## 📚 MODULE 9: Async Thunk Real-World Example — Complete API Integration

---

### 🧠 9.1 Concept Explanation — Real CRUD App

Abhi tak theory dekhi. Ab ek complete, real-world example banate hain:

**Project:** Products Management App
- JSONPlaceholder API use karenge (free, no setup needed)
- Fetch, Create, Update, Delete — sab thunks
- Proper loading states, error handling

---

### 💻 9.2 Complete Working Example — Products App

**Folder Structure:**
```
src/
├── app/store.js
├── features/
│   └── products/
│       ├── productsSlice.js   ← Slice + all thunks
│       ├── ProductsList.jsx   ← Main list component
│       ├── ProductForm.jsx    ← Create/Edit form
│       └── ProductItem.jsx    ← Single product
└── App.jsx
```

```javascript
// src/app/store.js
import { configureStore } from '@reduxjs/toolkit';
import productsReducer from '../features/products/productsSlice';

export const store = configureStore({
  reducer: {
    products: productsReducer,
  }
});
```

```javascript
// src/features/products/productsSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Base URL — .env se lo production mein
const API_BASE = 'https://jsonplaceholder.typicode.com';

// ============================================================
// ASYNC THUNKS — Sab API calls yahan
// ============================================================

// ---- 1. Sab products fetch karo ----
export const fetchProducts = createAsyncThunk(
  'products/fetchAll',
  async (_, thunkAPI) => {
    const response = await fetch(`${API_BASE}/posts?_limit=10`);
    // posts ko products ki jagah use kar rahe hain (demo ke liye)
    if (!response.ok) {
      return thunkAPI.rejectWithValue('Products load nahi hue. Try again.');
    }
    return response.json();
  }
);

// ---- 2. Single product fetch karo ----
export const fetchProductById = createAsyncThunk(
  'products/fetchById',
  async (productId, thunkAPI) => {
    const response = await fetch(`${API_BASE}/posts/${productId}`);
    if (!response.ok) {
      return thunkAPI.rejectWithValue(`Product ${productId} nahi mila`);
    }
    return response.json();
  }
);

// ---- 3. Naya product create karo ----
export const createProduct = createAsyncThunk(
  'products/create',
  async (productData, thunkAPI) => {
    const response = await fetch(`${API_BASE}/posts`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(productData)
    });
    if (!response.ok) {
      return thunkAPI.rejectWithValue('Product create nahi hua');
    }
    const newProduct = await response.json();
    return newProduct; // Server se naya product (with id)
  }
);

// ---- 4. Product update karo ----
export const updateProduct = createAsyncThunk(
  'products/update',
  async ({ id, ...changes }, thunkAPI) => {
    // Destructuring: id alag, baaki changes mein
    const response = await fetch(`${API_BASE}/posts/${id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ id, ...changes })
    });
    if (!response.ok) {
      return thunkAPI.rejectWithValue('Update fail hua');
    }
    return response.json(); // Updated product
  }
);

// ---- 5. Product delete karo ----
export const deleteProduct = createAsyncThunk(
  'products/delete',
  async (productId, thunkAPI) => {
    const response = await fetch(`${API_BASE}/posts/${productId}`, {
      method: 'DELETE'
    });
    if (!response.ok) {
      return thunkAPI.rejectWithValue('Delete fail hua');
    }
    return productId; // Deleted ID return karo (state update ke liye)
  }
);

// ============================================================
// SLICE
// ============================================================
const productsSlice = createSlice({
  name: 'products',
  initialState: {
    items: [],
    selectedProduct: null,
    loading: false,
    submitting: false, // Create/Update/Delete ke liye alag loading
    error: null,
    successMessage: null,
  },
  
  reducers: {
    // Sync actions
    clearMessages(state) {
      state.error = null;
      state.successMessage = null;
    },
    selectProduct(state, action) {
      state.selectedProduct = action.payload;
    },
    clearSelectedProduct(state) {
      state.selectedProduct = null;
    }
  },
  
  extraReducers: (builder) => {
    // ---- fetchProducts ----
    builder
      .addCase(fetchProducts.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchProducts.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchProducts.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      })

    // ---- fetchProductById ----
    .addCase(fetchProductById.pending, (state) => {
      state.loading = true;
    })
    .addCase(fetchProductById.fulfilled, (state, action) => {
      state.loading = false;
      state.selectedProduct = action.payload;
    })
    .addCase(fetchProductById.rejected, (state, action) => {
      state.loading = false;
      state.error = action.payload;
    })

    // ---- createProduct ----
    .addCase(createProduct.pending, (state) => {
      state.submitting = true; // Create ke liye submitting use karo
      state.error = null;
    })
    .addCase(createProduct.fulfilled, (state, action) => {
      state.submitting = false;
      state.items.unshift(action.payload); // Nayi product ko list mein add karo (top pe)
      state.successMessage = 'Product successfully create hua!';
    })
    .addCase(createProduct.rejected, (state, action) => {
      state.submitting = false;
      state.error = action.payload;
    })

    // ---- updateProduct ----
    .addCase(updateProduct.pending, (state) => {
      state.submitting = true;
      state.error = null;
    })
    .addCase(updateProduct.fulfilled, (state, action) => {
      state.submitting = false;
      // Updated product ko list mein replace karo
      const index = state.items.findIndex(item => item.id === action.payload.id);
      if (index !== -1) {
        state.items[index] = action.payload;
      }
      state.successMessage = 'Product update hua!';
    })
    .addCase(updateProduct.rejected, (state, action) => {
      state.submitting = false;
      state.error = action.payload;
    })

    // ---- deleteProduct ----
    .addCase(deleteProduct.pending, (state) => {
      state.submitting = true;
    })
    .addCase(deleteProduct.fulfilled, (state, action) => {
      state.submitting = false;
      // Delete hua product ID se filter karo
      state.items = state.items.filter(item => item.id !== action.payload);
      state.successMessage = 'Product delete hua!';
    })
    .addCase(deleteProduct.rejected, (state, action) => {
      state.submitting = false;
      state.error = action.payload;
    });
  }
});

export const { clearMessages, selectProduct, clearSelectedProduct } = productsSlice.actions;

// Selectors
export const selectAllProducts = (state) => state.products.items;
export const selectSelectedProduct = (state) => state.products.selectedProduct;
export const selectProductsLoading = (state) => state.products.loading;
export const selectProductsSubmitting = (state) => state.products.submitting;
export const selectProductsError = (state) => state.products.error;
export const selectProductsSuccess = (state) => state.products.successMessage;

export default productsSlice.reducer;
```

```javascript
// src/features/products/ProductsList.jsx
import React, { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import {
  fetchProducts,
  deleteProduct,
  selectAllProducts,
  selectProductsLoading,
  selectProductsError,
  clearMessages
} from './productsSlice';

function ProductsList() {
  const dispatch = useDispatch();
  const products = useSelector(selectAllProducts);
  const loading = useSelector(selectProductsLoading);
  const error = useSelector(selectProductsError);

  useEffect(() => {
    dispatch(fetchProducts());
    // Cleanup — component unmount pe messages clear karo
    return () => dispatch(clearMessages());
  }, [dispatch]);

  const handleDelete = (id) => {
    if (window.confirm('Kya aap sure hain delete karna chahte hain?')) {
      dispatch(deleteProduct(id));
    }
  };

  if (loading) return <div className="loading">Loading products...</div>;
  if (error) return (
    <div className="error">
      <p>Error: {error}</p>
      <button onClick={() => dispatch(fetchProducts())}>Dobara try karo</button>
    </div>
  );

  return (
    <div>
      <h2>Products List ({products.length})</h2>
      {products.length === 0 ? (
        <p>Koi product nahi mila</p>
      ) : (
        <ul>
          {products.map(product => (
            <li key={product.id} style={{ marginBottom: '10px', borderBottom: '1px solid #eee' }}>
              <strong>{product.title}</strong>
              <p>{product.body?.slice(0, 80)}...</p>
              <button onClick={() => handleDelete(product.id)} style={{ color: 'red' }}>
                Delete
              </button>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}

export default ProductsList;
```

```javascript
// src/features/products/ProductForm.jsx
import React, { useState } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import {
  createProduct,
  selectProductsSubmitting,
  selectProductsError,
  selectProductsSuccess,
  clearMessages
} from './productsSlice';

function ProductForm() {
  const dispatch = useDispatch();
  const submitting = useSelector(selectProductsSubmitting);
  const error = useSelector(selectProductsError);
  const successMessage = useSelector(selectProductsSuccess);
  
  const [formData, setFormData] = useState({
    title: '',
    body: '',
    userId: 1
  });

  const handleChange = (e) => {
    setFormData(prev => ({
      ...prev,
      [e.target.name]: e.target.value
    }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    if (!formData.title.trim()) return;
    
    const result = await dispatch(createProduct(formData));
    
    // createProduct.fulfilled check karo
    if (createProduct.fulfilled.match(result)) {
      // Success! Form reset karo
      setFormData({ title: '', body: '', userId: 1 });
    }
  };

  return (
    <form onSubmit={handleSubmit} style={{ marginBottom: '20px' }}>
      <h3>Naya Product Add Karo</h3>
      
      {error && <p style={{ color: 'red' }}>{error}</p>}
      {successMessage && <p style={{ color: 'green' }}>{successMessage}</p>}
      
      <div>
        <input
          name="title"
          value={formData.title}
          onChange={handleChange}
          placeholder="Product title"
          required
          style={{ display: 'block', marginBottom: '10px', width: '300px' }}
        />
        <textarea
          name="body"
          value={formData.body}
          onChange={handleChange}
          placeholder="Description"
          style={{ display: 'block', marginBottom: '10px', width: '300px', height: '80px' }}
        />
        <button type="submit" disabled={submitting}>
          {submitting ? 'Creating...' : 'Product Create Karo'}
        </button>
      </div>
    </form>
  );
}

export default ProductForm;
```

```javascript
// src/App.jsx
import React from 'react';
import ProductForm from './features/products/ProductForm';
import ProductsList from './features/products/ProductsList';

function App() {
  return (
    <div style={{ maxWidth: '600px', margin: '20px auto', fontFamily: 'sans-serif' }}>
      <h1>Products Manager (Redux + Thunk)</h1>
      <ProductForm />
      <ProductsList />
    </div>
  );
}

export default App;
```

---

### 💡 9.3 Pro Tips — Real App Mein

1. **Loading states alag rakho:** `loading` (fetch ke liye) vs `submitting` (create/update/delete ke liye) — dono ko mix mat karo
2. **Error messages user-friendly rakho:** Technical error messages user ko mat dikhao
3. **Optimistic updates** — Server response ka intezaar mat karo, pehle UI update karo, phir rollback agar fail ho (RTK Query mein easier)
4. **AbortController use karo** — Component unmount pe pending requests cancel karo

```javascript
// AbortController example
useEffect(() => {
  const promise = dispatch(fetchProducts());
  
  // Cleanup — component unmount pe cancel
  return () => promise.abort();
}, [dispatch]);
```

---

### 🎯 9.4 Interview Questions — Module 9

**Q1: loading aur submitting states alag kyun rakhte hain?**  
**A:** Fetch ke time alag UI dikhate hain (skeleton/spinner), aur create/update ke time form ko disable karte hain aur "Saving..." dikhate hain. Alag states se granular control milti hai.

**Q2: createAsyncThunk ka return value kaise check karte hain component mein?**  
**A:** `dispatch(thunk()).then()` ya `await dispatch(thunk())` aur phir `thunk.fulfilled.match(result)` se check karo.

**Q3: API base URL .env mein kyun rakhte hain?**  
**A:** Different environments ke liye different URLs hoti hain (development, staging, production). `.env` mein rakhne se code change kiye bina environment switch ho sakta hai.

---

### 📋 9.5 Practice Task

Poora **Student Management CRUD App** banao:
1. Fetch all students (`GET /api/students`)
2. Create student (`POST /api/students`)
3. Update student (`PUT /api/students/:id`)  
4. Delete student (`DELETE /api/students/:id`)

JSONPlaceholder use karo (users endpoint):
- Base URL: `https://jsonplaceholder.typicode.com`
- Students = `/users` endpoint

Proper loading, error, success states dono fetch aur submit operations mein.
