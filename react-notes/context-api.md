# Introduction to React Context API

As a newcomer to ReactJS, you've likely encountered state and props for managing and passing data between components. However, as your application grows, passing props through multiple levels (known as "prop drilling") can become cumbersome. The **Context API** offers a solution by providing a way to share values between components without explicitly passing props through every level of the component tree.

In this write-up, we'll cover:

1. **What is the Context API?**
2. **Why Use the Context API?**
3. **How to Use the Context API**
4. **Step-by-Step Examples**
5. **Common Pitfalls**
6. **Building a Sample Project**
   - Setting up with Vite
   - Using `useState` and `useEffect`
   - Styling with React Bootstrap
   - Fetching Data with Axios
   - Implementing Context API
7. **Conclusion**

Let's dive in!

---

## 1. What is the Context API?

The **Context API** is a feature in React that allows you to share state (data) across your entire app (or part of it) without passing props down manually at every level.

- **Global State Management**: Provides a way to create global variables that can be passed around.
- **Avoids Prop Drilling**: Eliminates the need to pass props through intermediate components.
- **Use Cases**: Ideal for theming, user authentication, locale settings, and any data that needs to be accessible by many components at different nesting levels.

---

## 2. Why Use the Context API?

### Avoiding Prop Drilling

In a typical React application, data is passed top-down (parent to child) via props. This can become cumbersome when many components need the same data, especially if they are deeply nested.

**Example of Prop Drilling**:

```jsx
function App() {
  const user = { name: 'Alice' };
  return <Parent user={user} />;
}

function Parent({ user }) {
  return <Child user={user} />;
}

function Child({ user }) {
  return <Grandchild user={user} />;
}

function Grandchild({ user }) {
  return <p>User: {user.name}</p>;
}
```

Here, the `user` prop is passed through multiple components that don't use it, just to reach `Grandchild`.

### Simplifying State Sharing

With the Context API, you can provide the `user` data at a higher level and consume it directly where needed, without intermediate components needing to pass it along.

**Using Context API**:

```jsx
// Create Context
const UserContext = React.createContext();

function App() {
  const user = { name: 'Alice' };
  return (
    <UserContext.Provider value={user}>
      <Parent />
    </UserContext.Provider>
  );
}

function Parent() {
  return <Child />;
}

function Child() {
  return <Grandchild />;
}

function Grandchild() {
  const user = useContext(UserContext);
  return <p>User: {user.name}</p>;
}
```

---

## 3. How to Use the Context API

Using the Context API involves three main steps:

1. **Creating a Context**
2. **Providing the Context**
3. **Consuming the Context**

### Step 1: Creating a Context

Use `React.createContext()` to create a new context.

```jsx
import React from 'react';

const MyContext = React.createContext(defaultValue);
```

- **defaultValue**: Optional. Used when a component consumes the context but does not have a matching Provider above it in the tree.

### Step 2: Providing the Context

Wrap your component tree with the Context Provider and pass the value you want to share.

```jsx
<MyContext.Provider value={sharedData}>
  {/* Components that can access sharedData */}
</MyContext.Provider>
```

### Step 3: Consuming the Context

Components can consume the context in two ways:

- **Using `useContext` Hook** (Functional Components):

  ```jsx
  import { useContext } from 'react';

  function MyComponent() {
    const contextData = useContext(MyContext);
    return <div>{contextData}</div>;
  }
  ```

- **Using `Context.Consumer`** (Class Components or when you need to pass a function as a child):

  ```jsx
  <MyContext.Consumer>
    {(contextData) => <div>{contextData}</div>}
  </MyContext.Consumer>
  ```

---

## 4. Step-by-Step Examples

### Example 1: Theme Context

Let's create a simple example where we manage a theme (light or dark) using the Context API.

#### Step 1: Create a Theme Context

```jsx
// ThemeContext.js
import React from 'react';

const ThemeContext = React.createContext('light'); // Default value is 'light'

export default ThemeContext;
```

#### Step 2: Provide the Theme Context

```jsx
// App.jsx
import React, { useState } from 'react';
import ThemeContext from './ThemeContext';
import Toolbar from './Toolbar';

function App() {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme((current) => (current === 'light' ? 'dark' : 'light'));
  };

  return (
    <ThemeContext.Provider value={theme}>
      <Toolbar toggleTheme={toggleTheme} />
    </ThemeContext.Provider>
  );
}

export default App;
```

#### Step 3: Consume the Theme Context

```jsx
// Toolbar.jsx
import React from 'react';
import ThemedButton from './ThemedButton';

function Toolbar({ toggleTheme }) {
  return (
    <div>
      <ThemedButton />
      <button onClick={toggleTheme}>Toggle Theme</button>
    </div>
  );
}

export default Toolbar;
```

```jsx
// ThemedButton.jsx
import React, { useContext } from 'react';
import ThemeContext from './ThemeContext';

function ThemedButton() {
  const theme = useContext(ThemeContext);

  const style = {
    backgroundColor: theme === 'light' ? '#eee' : '#333',
    color: theme === 'light' ? '#000' : '#fff',
    padding: '10px',
    margin: '10px',
  };

  return <button style={style}>I am styled by theme context!</button>;
}

export default ThemedButton;
```

### Example 2: User Authentication Context

Let's create an authentication context to manage user login state.

#### Step 1: Create Auth Context

```jsx
// AuthContext.js
import React from 'react';

const AuthContext = React.createContext();

export default AuthContext;
```

#### Step 2: Provide Auth Context

```jsx
// App.jsx
import React, { useState } from 'react';
import AuthContext from './AuthContext';
import LoginPage from './LoginPage';
import Dashboard from './Dashboard';

function App() {
  const [user, setUser] = useState(null);

  const login = (username) => setUser({ name: username });
  const logout = () => setUser(null);

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {user ? <Dashboard /> : <LoginPage />}
    </AuthContext.Provider>
  );
}

export default App;
```

#### Step 3: Consume Auth Context

```jsx
// LoginPage.jsx
import React, { useContext, useState } from 'react';
import AuthContext from './AuthContext';

function LoginPage() {
  const [username, setUsername] = useState('');
  const { login } = useContext(AuthContext);

  const handleSubmit = (e) => {
    e.preventDefault();
    login(username);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type='text'
        value={username}
        onChange={(e) => setUsername(e.target.value)}
        placeholder='Enter username'
      />
      <button type='submit'>Login</button>
    </form>
  );
}

export default LoginPage;
```

```jsx
// Dashboard.jsx
import React, { useContext } from 'react';
import AuthContext from './AuthContext';

function Dashboard() {
  const { user, logout } = useContext(AuthContext);

  return (
    <div>
      <h1>Welcome, {user.name}!</h1>
      <button onClick={logout}>Logout</button>
    </div>
  );
}

export default Dashboard;
```

### Example 3: Counter Context

In this example, we'll create a simple counter application where multiple components can display and update the same counter value using the Context API.

#### Overview

- **Objective**: Manage a counter value globally without prop drilling.
- **Components**:
  - **App**: Provides the CounterContext.
  - **CounterDisplay**: Displays the current counter value.
  - **CounterControls**: Contains buttons to increment and decrement the counter.

#### Step 1: Create a Counter Context

First, we'll create a context for our counter.

```jsx
// CounterContext.js
import React from 'react';

const CounterContext = React.createContext();

export default CounterContext;
```

#### Step 2: Provide the Counter Context

Next, we'll set up our `App` component to provide the counter context to its child components.

```jsx
// App.jsx
import React, { useState } from 'react';
import CounterContext from './CounterContext';
import CounterDisplay from './CounterDisplay';
import CounterControls from './CounterControls';

function App() {
  const [count, setCount] = useState(0);

  // Context value includes both the count and functions to modify it
  const counterContextValue = {
    count,
    increment: () => setCount((prev) => prev + 1),
    decrement: () => setCount((prev) => prev - 1),
  };

  return (
    <CounterContext.Provider value={counterContextValue}>
      <div style={{ textAlign: 'center', marginTop: '50px' }}>
        <h1>Counter App Using Context API</h1>
        <CounterDisplay />
        <CounterControls />
      </div>
    </CounterContext.Provider>
  );
}

export default App;
```

**Explanation**:

- We use `useState` to manage the counter value.
- The `CounterContext.Provider` provides an object containing:
  - `count`: The current counter value.
  - `increment`: Function to increase the counter.
  - `decrement`: Function to decrease the counter.
- Child components (`CounterDisplay` and `CounterControls`) can access this context.

#### Step 3: Consume the Counter Context

We'll create components to display and control the counter.

##### CounterDisplay Component

```jsx
// CounterDisplay.jsx
import React, { useContext } from 'react';
import CounterContext from './CounterContext';

function CounterDisplay() {
  const { count } = useContext(CounterContext);

  return (
    <div>
      <h2>Current Count: {count}</h2>
    </div>
  );
}

export default CounterDisplay;
```

**Explanation**:

- We use `useContext` to access the `count` from `CounterContext`.
- Displays the current counter value.

##### CounterControls Component

```jsx
// CounterControls.jsx
import React, { useContext } from 'react';
import CounterContext from './CounterContext';

function CounterControls() {
  const { increment, decrement } = useContext(CounterContext);

  return (
    <div>
      <button onClick={decrement} style={{ marginRight: '10px' }}>
        Decrement
      </button>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

export default CounterControls;
```

**Explanation**:

- Accesses `increment` and `decrement` functions from `CounterContext`.
- Provides buttons to modify the counter value.

#### Full Code for Reference

**App.jsx**

```jsx
import React, { useState } from 'react';
import CounterContext from './CounterContext';
import CounterDisplay from './CounterDisplay';
import CounterControls from './CounterControls';

function App() {
  const [count, setCount] = useState(0);

  const counterContextValue = {
    count,
    increment: () => setCount((prev) => prev + 1),
    decrement: () => setCount((prev) => prev - 1),
  };

  return (
    <CounterContext.Provider value={counterContextValue}>
      <div style={{ textAlign: 'center', marginTop: '50px' }}>
        <h1>Counter App Using Context API</h1>
        <CounterDisplay />
        <CounterControls />
      </div>
    </CounterContext.Provider>
  );
}

export default App;
```

**CounterContext.js**

```jsx
import React from 'react';

const CounterContext = React.createContext();

export default CounterContext;
```

**CounterDisplay.jsx**

```jsx
import React, { useContext } from 'react';
import CounterContext from './CounterContext';

function CounterDisplay() {
  const { count } = useContext(CounterContext);

  return (
    <div>
      <h2>Current Count: {count}</h2>
    </div>
  );
}

export default CounterDisplay;
```

**CounterControls.jsx**

```jsx
import React, { useContext } from 'react';
import CounterContext from './CounterContext';

function CounterControls() {
  const { increment, decrement } = useContext(CounterContext);

  return (
    <div>
      <button onClick={decrement} style={{ marginRight: '10px' }}>
        Decrement
      </button>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

export default CounterControls;
```

#### Running the Counter Example

1. **Create a New Project**: If you haven't already, set up a new React project using Vite:

   ```bash
   npm create vite@latest counter-app -- --template react
   cd counter-app
   npm install
   ```

2. **Add the Components**: Create the files as shown above in the `src/` directory.

3. **Start the Development Server**:

   ```bash
   npm run dev
   ```

4. **View the App**: Open the provided URL (e.g., `http://localhost:5173`) to see the counter app in action.

---

### Example 4: Shopping Cart Context

Let's build a slightly more complex example—a shopping cart where we can add and remove items, and the cart's state is managed using Context API.

#### Overview

- **Objective**: Manage a shopping cart's state globally.
- **Components**:
  - **App**: Provides the CartContext.
  - **ProductList**: Displays a list of products that can be added to the cart.
  - **Cart**: Displays items in the cart and allows removing them.
  - **CartContext**: Manages cart state.

#### Step 1: Create Cart Context

```jsx
// CartContext.js
import React from 'react';

const CartContext = React.createContext();

export default CartContext;
```

#### Step 2: Provide the Cart Context

```jsx
// App.jsx
import React, { useState } from 'react';
import CartContext from './CartContext';
import ProductList from './ProductList';
import Cart from './Cart';
import { Container, Row, Col } from 'react-bootstrap';

function App() {
  const [cartItems, setCartItems] = useState([]);

  const addToCart = (product) => {
    setCartItems((prevItems) => [...prevItems, product]);
  };

  const removeFromCart = (productId) => {
    setCartItems((prevItems) =>
      prevItems.filter((item) => item.id !== productId)
    );
  };

  const cartContextValue = {
    cartItems,
    addToCart,
    removeFromCart,
  };

  return (
    <CartContext.Provider value={cartContextValue}>
      <Container className='mt-4'>
        <Row>
          <Col md={8}>
            <h2>Products</h2>
            <ProductList />
          </Col>
          <Col md={4}>
            <h2>Shopping Cart</h2>
            <Cart />
          </Col>
        </Row>
      </Container>
    </CartContext.Provider>
  );
}

export default App;
```

**Explanation**:

- Manages `cartItems` state.
- Provides `addToCart` and `removeFromCart` functions.
- Wraps child components with `CartContext.Provider`.

#### Step 3: Consume the Cart Context

##### ProductList Component

```jsx
// ProductList.jsx
import React, { useContext } from 'react';
import CartContext from './CartContext';
import { Button, Card } from 'react-bootstrap';

function ProductList() {
  const { addToCart } = useContext(CartContext);

  const products = [
    { id: 1, name: 'Apple', price: 0.99 },
    { id: 2, name: 'Orange', price: 0.79 },
    { id: 3, name: 'Banana', price: 0.49 },
  ];

  return (
    <>
      {products.map((product) => (
        <Card key={product.id} className='mb-2'>
          <Card.Body>
            <Card.Title>{product.name}</Card.Title>
            <Card.Text>Price: ${product.price.toFixed(2)}</Card.Text>
            <Button onClick={() => addToCart(product)}>Add to Cart</Button>
          </Card.Body>
        </Card>
      ))}
    </>
  );
}

export default ProductList;
```

**Explanation**:

- Displays a list of products.
- Uses `addToCart` from `CartContext` to add products to the cart.

##### Cart Component

```jsx
// Cart.jsx
import React, { useContext } from 'react';
import CartContext from './CartContext';
import { ListGroup, Button } from 'react-bootstrap';

function Cart() {
  const { cartItems, removeFromCart } = useContext(CartContext);

  return (
    <ListGroup>
      {cartItems.length === 0 ? (
        <ListGroup.Item>Your cart is empty.</ListGroup.Item>
      ) : (
        cartItems.map((item) => (
          <ListGroup.Item key={item.id}>
            {item.name} - ${item.price.toFixed(2)}
            <Button
              variant='danger'
              size='sm'
              onClick={() => removeFromCart(item.id)}
              style={{ float: 'right' }}
            >
              Remove
            </Button>
          </ListGroup.Item>
        ))
      )}
    </ListGroup>
  );
}

export default Cart;
```

**Explanation**:

- Displays items in the cart.
- Uses `removeFromCart` to remove items from the cart.

#### Full Code for Reference

**App.jsx**

```jsx
import React, { useState } from 'react';
import CartContext from './CartContext';
import ProductList from './ProductList';
import Cart from './Cart';
import { Container, Row, Col } from 'react-bootstrap';

function App() {
  const [cartItems, setCartItems] = useState([]);

  const addToCart = (product) => {
    setCartItems((prevItems) => [...prevItems, product]);
  };

  const removeFromCart = (productId) => {
    setCartItems((prevItems) =>
      prevItems.filter((item) => item.id !== productId)
    );
  };

  const cartContextValue = {
    cartItems,
    addToCart,
    removeFromCart,
  };

  return (
    <CartContext.Provider value={cartContextValue}>
      <Container className='mt-4'>
        <Row>
          <Col md={8}>
            <h2>Products</h2>
            <ProductList />
          </Col>
          <Col md={4}>
            <h2>Shopping Cart</h2>
            <Cart />
          </Col>
        </Row>
      </Container>
    </CartContext.Provider>
  );
}

export default App;
```

**CartContext.js**

```jsx
import React from 'react';

const CartContext = React.createContext();

export default CartContext;
```

**ProductList.jsx**

```jsx
import React, { useContext } from 'react';
import CartContext from './CartContext';
import { Button, Card } from 'react-bootstrap';

function ProductList() {
  const { addToCart } = useContext(CartContext);

  const products = [
    { id: 1, name: 'Apple', price: 0.99 },
    { id: 2, name: 'Orange', price: 0.79 },
    { id: 3, name: 'Banana', price: 0.49 },
  ];

  return (
    <>
      {products.map((product) => (
        <Card key={product.id} className='mb-2'>
          <Card.Body>
            <Card.Title>{product.name}</Card.Title>
            <Card.Text>Price: ${product.price.toFixed(2)}</Card.Text>
            <Button onClick={() => addToCart(product)}>Add to Cart</Button>
          </Card.Body>
        </Card>
      ))}
    </>
  );
}

export default ProductList;
```

**Cart.jsx**

```jsx
import React, { useContext } from 'react';
import CartContext from './CartContext';
import { ListGroup, Button } from 'react-bootstrap';

function Cart() {
  const { cartItems, removeFromCart } = useContext(CartContext);

  return (
    <ListGroup>
      {cartItems.length === 0 ? (
        <ListGroup.Item>Your cart is empty.</ListGroup.Item>
      ) : (
        cartItems.map((item) => (
          <ListGroup.Item key={item.id}>
            {item.name} - ${item.price.toFixed(2)}
            <Button
              variant='danger'
              size='sm'
              onClick={() => removeFromCart(item.id)}
              style={{ float: 'right' }}
            >
              Remove
            </Button>
          </ListGroup.Item>
        ))
      )}
    </ListGroup>
  );
}

export default Cart;
```

#### Running the Shopping Cart Example

1. **Create a New Project**:

   ```bash
   npm create vite@latest shopping-cart -- --template react
   cd shopping-cart
   npm install
   ```

2. **Install React Bootstrap**:

   ```bash
   npm install react-bootstrap bootstrap
   ```

3. **Import Bootstrap CSS in `main.jsx`**:

   ```jsx
   // main.jsx
   import React from 'react';
   import ReactDOM from 'react-dom/client';
   import App from './App';
   import 'bootstrap/dist/css/bootstrap.min.css';

   ReactDOM.createRoot(document.getElementById('root')).render(
     <React.StrictMode>
       <App />
     </React.StrictMode>
   );
   ```

4. **Add the Components**: Create the files as shown above in the `src/` directory.

5. **Start the Development Server**:

   ```bash
   npm run dev
   ```

6. **View the App**: Open the provided URL to see the shopping cart app.

---

## 5. Common Pitfalls

### Overusing Context

- **Avoid Using Context for Everything**: Context is not a replacement for props or state in components. Use it when you need to share data widely.
- **Complex State Management**: For complex state updates, consider state management libraries like Redux or MobX.

### Performance Issues

- **Unnecessary Re-renders**: When context value changes, all components consuming that context will re-render.
- **Optimization Techniques**:
  - **Split Contexts**: Use multiple contexts for different pieces of data.
  - **Memoization**: Use `React.memo` and `useMemo` to prevent unnecessary re-renders.

### Mutating Context Values

- **Do Not Mutate Context**: Always provide a new value to the context provider to trigger updates.

```jsx
// Incorrect: Mutating existing context value
value.someProperty = newValue;
```

- **Correct**: Create a new object when updating context value.

```jsx
setValue((prev) => ({ ...prev, someProperty: newValue }));
```

---

## 6. Building a Sample Project

Let's build a simple **User Management** application using:

- **ReactJS with Vite**: For setting up the project.
- **`useState` and `useEffect`**: For state and side effects.
- **React Bootstrap**: For styling.
- **Axios**: For making HTTP requests.
- **Context API**: For managing global state.

### 6.1 Setting Up with Vite

#### Step 1: Create a New React Project with Vite

```bash
npm create vite@latest user-management -- --template react
```

#### Step 2: Navigate to the Project Directory

```bash
cd user-management
```

#### Step 3: Install Dependencies

```bash
npm install
```

#### Step 4: Install Additional Libraries

```bash
npm install react-bootstrap bootstrap axios
```

- **react-bootstrap**: React components for Bootstrap.
- **bootstrap**: Bootstrap CSS framework.
- **axios**: HTTP client for making API requests.

### 6.2 Project Structure

Your project structure should look like this:

```
user-management/
├── node_modules/
├── public/
├── src/
│   ├── components/
│   ├── context/
│   ├── pages/
│   ├── App.jsx
│   ├── main.jsx
├── package.json
├── vite.config.js
```

### 6.3 Setting Up Bootstrap

In `main.jsx`, import Bootstrap CSS:

```jsx
// main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import 'bootstrap/dist/css/bootstrap.min.css'; // Import Bootstrap CSS

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

### 6.4 Creating User Context

#### Step 1: Create Context

```jsx
// src/context/UserContext.jsx
import React from 'react';

const UserContext = React.createContext();

export default UserContext;
```

#### Step 2: Create Context Provider

```jsx
// src/context/UserProvider.jsx
import React, { useState, useEffect } from 'react';
import UserContext from './UserContext';
import axios from 'axios';

function UserProvider({ children }) {
  const [users, setUsers] = useState([]);

  // Fetch users from API
  useEffect(() => {
    axios
      .get('https://jsonplaceholder.typicode.com/users')
      .then((response) => setUsers(response.data))
      .catch((error) => console.error('Error fetching users:', error));
  }, []);

  return (
    <UserContext.Provider value={{ users, setUsers }}>
      {children}
    </UserContext.Provider>
  );
}

export default UserProvider;
```

### 6.5 Setting Up the App Component

```jsx
// App.jsx
import React from 'react';
import UserProvider from './context/UserProvider';
import UserList from './components/UserList';
import { Container } from 'react-bootstrap';

function App() {
  return (
    <UserProvider>
      <Container className='my-4'>
        <h1>User Management</h1>
        <UserList />
      </Container>
    </UserProvider>
  );
}

export default App;
```

### 6.6 Creating the UserList Component

```jsx
// src/components/UserList.jsx
import React, { useContext } from 'react';
import UserContext from '../context/UserContext';
import { ListGroup } from 'react-bootstrap';
import UserDetails from './UserDetails';

function UserList() {
  const { users } = useContext(UserContext);

  return (
    <ListGroup>
      {users.map((user) => (
        <ListGroup.Item key={user.id}>
          <UserDetails user={user} />
        </ListGroup.Item>
      ))}
    </ListGroup>
  );
}

export default UserList;
```

### 6.7 Creating the UserDetails Component

```jsx
// src/components/UserDetails.jsx
import React from 'react';
import { Card } from 'react-bootstrap';

function UserDetails({ user }) {
  return (
    <Card>
      <Card.Body>
        <Card.Title>{user.name}</Card.Title>
        <Card.Subtitle>{user.email}</Card.Subtitle>
        <Card.Text>
          <strong>Company:</strong> {user.company.name}
        </Card.Text>
      </Card.Body>
    </Card>
  );
}

export default UserDetails;
```

### 6.8 Adding Functionality to Add and Remove Users

#### Update UserProvider to Include Add and Remove Functions

```jsx
// src/context/UserProvider.jsx
function UserProvider({ children }) {
  const [users, setUsers] = useState([]);

  // Fetch users from API
  useEffect(() => {
    axios
      .get('https://jsonplaceholder.typicode.com/users')
      .then((response) => setUsers(response.data))
      .catch((error) => console.error('Error fetching users:', error));
  }, []);

  const addUser = (user) => {
    setUsers((prevUsers) => [...prevUsers, user]);
  };

  const removeUser = (id) => {
    setUsers((prevUsers) => prevUsers.filter((user) => user.id !== id));
  };

  return (
    <UserContext.Provider value={{ users, addUser, removeUser }}>
      {children}
    </UserContext.Provider>
  );
}
```

#### Create AddUser Component

```jsx
// src/components/AddUser.jsx
import React, { useState, useContext } from 'react';
import UserContext from '../context/UserContext';
import { Form, Button } from 'react-bootstrap';

function AddUser() {
  const { addUser } = useContext(UserContext);
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    const newUser = {
      id: Date.now(),
      name,
      email,
      company: { name: 'New Company' },
    };
    addUser(newUser);
    setName('');
    setEmail('');
  };

  return (
    <Form onSubmit={handleSubmit} className='my-4'>
      <Form.Group controlId='formName'>
        <Form.Label>Name</Form.Label>
        <Form.Control
          type='text'
          value={name}
          onChange={(e) => setName(e.target.value)}
          placeholder='Enter name'
          required
        />
      </Form.Group>
      <Form.Group controlId='formEmail' className='mt-2'>
        <Form.Label>Email</Form.Label>
        <Form.Control
          type='email'
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          placeholder='Enter email'
          required
        />
      </Form.Group>
      <Button variant='primary' type='submit' className='mt-3'>
        Add User
      </Button>
    </Form>
  );
}

export default AddUser;
```

#### Update App Component to Include AddUser

```jsx
// App.jsx
import React from 'react';
import UserProvider from './context/UserProvider';
import UserList from './components/UserList';
import AddUser from './components/AddUser';
import { Container } from 'react-bootstrap';

function App() {
  return (
    <UserProvider>
      <Container className='my-4'>
        <h1>User Management</h1>
        <AddUser />
        <UserList />
      </Container>
    </UserProvider>
  );
}

export default App;
```

#### Update UserDetails Component to Include Remove Functionality

```jsx
// src/components/UserDetails.jsx
import React, { useContext } from 'react';
import UserContext from '../context/UserContext';
import { Card, Button } from 'react-bootstrap';

function UserDetails({ user }) {
  const { removeUser } = useContext(UserContext);

  return (
    <Card>
      <Card.Body>
        <Card.Title>{user.name}</Card.Title>
        <Card.Subtitle>{user.email}</Card.Subtitle>
        <Card.Text>
          <strong>Company:</strong> {user.company.name}
        </Card.Text>
        <Button variant='danger' onClick={() => removeUser(user.id)}>
          Remove
        </Button>
      </Card.Body>
    </Card>
  );
}

export default UserDetails;
```

### 6.9 Running the Application

Start the development server:

```bash
npm run dev
```

Open the provided URL (usually `http://localhost:5173`) in your browser.

You should see a user management application where you can:

- View a list of users fetched from an API.
- Add a new user.
- Remove a user.

### 6.10 Project Recap

- **useState**: Used for managing component state (e.g., form inputs, users array).
- **useEffect**: Used for side effects like fetching data from an API.
- **React Bootstrap**: Used for styling components with Bootstrap.
- **Axios**: Used for making HTTP requests to fetch user data.
- **Context API**: Used to share user data and functions (`users`, `addUser`, `removeUser`) across components without prop drilling.

---

## 7. Key Takeaways

- **Context API** is useful for sharing state that needs to be accessed by multiple components at different levels of the component tree.
- **Create Context**: Use `React.createContext()` to create a context object.
- **Provide Context**: Wrap components with `Context.Provider` and pass the value you want to share.
- **Consume Context**: Use `useContext(Context)` to access the context value in functional components.

**Next Steps**:

- **Experiment**: Try extending the sample project with additional features like editing users or adding authentication.
- **Learn More**: Dive deeper into React hooks like `useReducer` for more complex state logic.
- **Explore Alternatives**: As your applications grow, consider learning about Redux or other state management libraries for handling complex state.
