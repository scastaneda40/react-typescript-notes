# TypeScript Patterns for React Cheat Sheet

This document provides quick reference examples for common TypeScript patterns when working with React.

## 1. Typing Component Props

Defining interfaces for component props is fundamental for type safety.

```typescript
import React from "react";

// Basic Props Interface
interface WelcomeMessageProps {
  name: string;
  enthusiasmLevel?: number; // Optional prop
}

const WelcomeMessage: React.FC<WelcomeMessageProps> = ({
  name,
  enthusiasmLevel = 1,
}) => {
  const getExclamationMarks = (num: number) => Array(num + 1).join("!");
  return (
    <h1>
      Hello, {name}
      {getExclamationMarks(enthusiasmLevel)}
    </h1>
  );
};

export default WelcomeMessage;

// Usage:
// <WelcomeMessage name="Alice" />
// <WelcomeMessage name="Bob" enthusiasmLevel={3} />

// Props with Children
interface CardProps {
  title: string;
  children: React.ReactNode; // For content passed between opening and closing tags
}

const Card: React.FC<CardProps> = ({ title, children }) => {
  return (
    <div className="border p-4 rounded shadow">
      <h2 className="text-xl font-bold mb-2">{title}</h2>
      <div>{children}</div>
    </div>
  );
};

// Usage:
// <Card title="My Card">
//   <p>This is some content inside the card.</p>
// </Card>
```

## 2. Typing State with `useState`

Ensuring your state variables are correctly typed.

```typescript
import React, { useState } from "react";

interface User {
  id: string;
  name: string;
  email: string;
}

const UserProfile: React.FC = () => {
  // Simple type
  const [count, setCount] = useState<number>(0);

  // Union type for initial null state
  const [currentUser, setCurrentUser] = useState<User | null>(null);

  // Array of objects
  const [users, setUsers] = useState<User[]>([]);

  // Object state
  const [formInput, setFormInput] = useState<{
    firstName: string;
    lastName: string;
  }>({
    firstName: "",
    lastName: "",
  });

  const handleLogin = () => {
    setCurrentUser({
      id: "u1",
      name: "John Doe",
      email: "john.doe@example.com",
    });
  };

  const handleAddUser = () => {
    setUsers((prevUsers) => [
      ...prevUsers,
      {
        id: `u${prevUsers.length + 1}`,
        name: "New User",
        email: "new@example.com",
      },
    ]);
  };

  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormInput((prev) => ({ ...prev, [name]: value }));
  };

  return (
    <div>
      <h2>useState with TypeScript</h2>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>

      {currentUser ? (
        <p>
          Logged in as: {currentUser.name} ({currentUser.email})
        </p>
      ) : (
        <button onClick={handleLogin}>Login</button>
      )}

      <h3>Users List</h3>
      <button onClick={handleAddUser}>Add User</button>
      <ul>
        {users.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>

      <h3>Form Input</h3>
      <input
        type="text"
        name="firstName"
        value={formInput.firstName}
        onChange={handleInputChange}
        placeholder="First Name"
      />
      <input
        type="text"
        name="lastName"
        value={formInput.lastName}
        onChange={handleInputChange}
        placeholder="Last Name"
      />
      <p>
        Full Name: {formInput.firstName} {formInput.lastName}
      </p>
    </div>
  );
};

export default UserProfile;
```

## 3. Typing Event Handlers

Correctly typing event objects for various DOM events.

```typescript
import React, { useState } from "react";

const EventHandlers: React.FC = () => {
  const [inputValue, setInputValue] = useState<string>("");
  const [checkboxChecked, setCheckboxChecked] = useState<boolean>(false);

  const handleButtonClick = (event: React.MouseEvent<HTMLButtonElement>) => {
    console.log("Button clicked!", event.currentTarget.tagName);
  };

  const handleInputChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    setInputValue(event.target.value);
  };

  const handleCheckboxChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    setCheckboxChecked(event.target.checked);
  };

  const handleFormSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault(); // Prevent default form submission
    alert(`Form submitted with input: ${inputValue}`);
  };

  const handleDivClick = (event: React.MouseEvent<HTMLDivElement>) => {
    console.log("Div clicked!", event.clientX, event.clientY);
  };

  return (
    <div className="p-4">
      <h2>Event Handlers with TypeScript</h2>

      <button
        onClick={handleButtonClick}
        className="bg-blue-500 text-white px-3 py-1 rounded mr-2"
      >
        Click Me
      </button>

      <form onSubmit={handleFormSubmit} className="my-4">
        <input
          type="text"
          value={inputValue}
          onChange={handleInputChange}
          placeholder="Type something..."
          className="border p-2 mr-2"
        />
        <button
          type="submit"
          className="bg-green-500 text-white px-3 py-1 rounded"
        >
          Submit
        </button>
        <p>Input Value: {inputValue}</p>
      </form>

      <label className="flex items-center mb-4">
        <input
          type="checkbox"
          checked={checkboxChecked}
          onChange={handleCheckboxChange}
          className="mr-2"
        />
        Check me
      </label>
      <p>Checkbox is {checkboxChecked ? "checked" : "unchecked"}</p>

      <div
        onClick={handleDivClick}
        className="border-2 border-dashed border-gray-400 p-8 mt-4 cursor-pointer"
      >
        Click this div to see coordinates in console
      </div>
    </div>
  );
};

export default EventHandlers;
```

## 4. Typing `useRef`

Using `useRef` for DOM elements and mutable values.

```typescript
import React, { useRef, useEffect } from "react";

const RefTyping: React.FC = () => {
  // For a DOM element (e.g., an input field)
  const inputRef = useRef<HTMLInputElement>(null);

  // For a generic mutable value that persists across renders
  const mutableValueRef = useRef<number>(0);

  useEffect(() => {
    if (inputRef.current) {
      inputRef.current.focus();
      console.log("Input element:", inputRef.current);
    }
    mutableValueRef.current = 100; // Set initial value
    console.log("mutableValueRef initialized to:", mutableValueRef.current);
  }, []);

  const handleFocusInput = () => {
    inputRef.current?.focus(); // Optional chaining for safety
  };

  const incrementMutableValue = () => {
    mutableValueRef.current += 1;
    console.log("Mutable value:", mutableValueRef.current);
    // Note: Changing mutableValueRef.current does NOT trigger a re-render
  };

  return (
    <div className="p-4">
      <h2>useRef with TypeScript</h2>
      <input
        type="text"
        ref={inputRef}
        placeholder="I will be focused"
        className="border p-2 mr-2"
      />
      <button
        onClick={handleFocusInput}
        className="bg-blue-500 text-white px-3 py-1 rounded"
      >
        Focus Input
      </button>

      <p className="mt-4">
        Mutable Value (check console for updates): {mutableValueRef.current}
      </p>
      <button
        onClick={incrementMutableValue}
        className="bg-green-500 text-white px-3 py-1 rounded"
      >
        Increment Mutable Value
      </button>
    </div>
  );
};

export default RefTyping;
```

## 5. Typing `useReducer`

Defining types for state and actions in `useReducer`.

```typescript
import React, { useReducer } from "react";

// 1. Define State Interface
interface CounterState {
  count: number;
}

// 2. Define Action Types
type CounterAction =
  | { type: "INCREMENT"; payload?: number }
  | { type: "DECREMENT"; payload?: number }
  | { type: "RESET" };

// 3. Create Reducer Function
const counterReducer = (
  state: CounterState,
  action: CounterAction
): CounterState => {
  switch (action.type) {
    case "INCREMENT":
      return { count: state.count + (action.payload || 1) };
    case "DECREMENT":
      return { count: state.count - (action.payload || 1) };
    case "RESET":
      return { count: 0 };
    default:
      // Ensure all action types are handled, or throw an error
      throw new Error("Unknown action type");
  }
};

const CounterWithReducer: React.FC = () => {
  // 4. Initialize useReducer
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <div className="p-4">
      <h2>useReducer with TypeScript</h2>
      <p>Count: {state.count}</p>
      <button
        onClick={() => dispatch({ type: "INCREMENT" })}
        className="bg-blue-500 text-white px-3 py-1 rounded mr-2"
      >
        Increment
      </button>
      <button
        onClick={() => dispatch({ type: "INCREMENT", payload: 5 })}
        className="bg-blue-500 text-white px-3 py-1 rounded mr-2"
      >
        Increment by 5
      </button>
      <button
        onClick={() => dispatch({ type: "DECREMENT" })}
        className="bg-red-500 text-white px-3 py-1 rounded mr-2"
      >
        Decrement
      </button>
      <button
        onClick={() => dispatch({ type: "RESET" })}
        className="bg-gray-500 text-white px-3 py-1 rounded"
      >
        Reset
      </button>
    </div>
  );
};

export default CounterWithReducer;
```

## 6. Typing Context API

Creating and consuming context with TypeScript.

```typescript
import React, { createContext, useContext, useState, ReactNode } from "react";

// 1. Define the shape of the Context Value
interface ThemeContextType {
  theme: "light" | "dark";
  toggleTheme: () => void;
}

// 2. Create the Context with a default value (or null and handle it)
// Using 'as ThemeContextType' asserts the type for the default value
const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// 3. Create a Provider Component
interface ThemeProviderProps {
  children: ReactNode;
}

const ThemeProvider: React.FC<ThemeProviderProps> = ({ children }) => {
  const [theme, setTheme] = useState<"light" | "dark">("light");

  const toggleTheme = () => {
    setTheme((prevTheme) => (prevTheme === "light" ? "dark" : "light"));
  };

  const contextValue: ThemeContextType = { theme, toggleTheme };

  return (
    <ThemeContext.Provider value={contextValue}>
      {children}
    </ThemeContext.Provider>
  );
};

// 4. Create a Custom Hook for consuming the Context
const useTheme = () => {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error("useTheme must be used within a ThemeProvider");
  }
  return context;
};

// Example Consumer Component
const ThemeSwitcher: React.FC = () => {
  const { theme, toggleTheme } = useTheme();

  return (
    <button
      onClick={toggleTheme}
      className={`px-4 py-2 rounded ${
        theme === "light"
          ? "bg-gray-200 text-gray-800"
          : "bg-gray-800 text-white"
      }`}
    >
      Switch to {theme === "light" ? "Dark" : "Light"} Mode
    </button>
  );
};

// Another Consumer Component
const ThemedContent: React.FC = () => {
  const { theme } = useTheme();
  return (
    <div
      className={`p-8 mt-4 rounded ${
        theme === "light" ? "bg-white text-black" : "bg-gray-900 text-white"
      }`}
    >
      <p>This content is styled with the {theme} theme.</p>
      <ThemeSwitcher />
    </div>
  );
};

// Usage in App.tsx or similar:
// import ThemeProvider from './ThemeProvider'; // Assuming ThemeProvider is in its own file
// import ThemedContent from './ThemedContent';
//
// const App = () => {
//   return (
//     <ThemeProvider>
//       <ThemedContent />
//     </ThemeProvider>
//   );
// };
```

## 7. Common Types

```typescript
onAdd: (title: string) => void;

onDelete: (id: string) => void;

onToggle: (id: string) => void;

onUpdate: (id: string, data: Partial<Task>) => void;

onSubmit: (form: FormData) => void;

onSearch: (query: string) => void;

onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;

onClick: (e: React.MouseEvent<HTMLButtonElement>) => void;
```
## 7. Common Utility

| Utility        | Description             | Example                |           |
| -------------- | ----------------------- | ---------------------- | --------- |
| `Partial<T>`   | All fields optional     | `Partial<Task>`        |           |
| `Pick<T, K>`   | Choose specific fields  | `Pick<Task, 'id'       | 'title'>` |
| `Omit<T, K>`   | Exclude specific fields | `Omit<Task, 'id'>`     |           |
| `Record<K, T>` | Object map              | `Record<string, Task>` |           |



