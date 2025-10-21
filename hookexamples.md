# React Hooks Examples Cheat Sheet

This document provides quick reference examples for common React Hooks.

## 1. `useState`

Manages state within functional components.

```typescript
import React, { useState } from "react";

const Counter: React.FC = () => {
  const [count, setCount] = useState<number>(0);
  const [message, setMessage] = useState<string>("Hello");
  const [user, setUser] = useState<{ id: string; name: string } | null>(null);

  const increment = () => setCount((prevCount) => prevCount + 1);
  const decrement = () => setCount((prevCount) => prevCount - 1);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>

      <p>Message: {message}</p>
      <button onClick={() => setMessage("Goodbye")}>Change Message</button>

      {user ? (
        <p>User: {user.name}</p>
      ) : (
        <button onClick={() => setUser({ id: "1", name: "Alice" })}>
          Set User
        </button>
      )}
    </div>
  );
};

export default Counter;
```

## 2. `useEffect`

Performs side effects in functional components (data fetching, subscriptions, manually changing the DOM).

```typescript
import React, { useState, useEffect } from "react";

const DataFetcher: React.FC = () => {
  const [data, setData] = useState<string | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);
  const [userId, setUserId] = useState<number>(1);

  // Effect for data fetching (runs on mount and when userId changes)
  useEffect(() => {
    setLoading(true);
    setError(null);
    setData(null);

    const fetchData = async () => {
      try {
        const response = await fetch(
          `https://jsonplaceholder.typicode.com/posts/${userId}`
        );
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const json = await response.json();
        setData(json.title);
      } catch (e: any) {
        setError(e.message);
      } finally {
        setLoading(false);
      }
    };

    fetchData();

    // Cleanup function (optional): runs on unmount or before re-running the effect
    return () => {
      console.log("Cleanup for userId:", userId);
      // e.g., cancel a subscription or a pending fetch request
    };
  }, [userId]); // Dependency array: re-run effect if userId changes

  // Effect for logging (runs only once on mount)
  useEffect(() => {
    console.log("Component mounted");
    return () => {
      console.log("Component unmounted");
    };
  }, []); // Empty dependency array: runs only once on mount and cleanup on unmount

  // Effect with no dependency array (runs on every render) - generally avoid this
  useEffect(() => {
    console.log("This effect runs on every render!");
  });

  return (
    <div>
      <h2>Data Fetcher</h2>
      <p>Current User ID: {userId}</p>
      <button onClick={() => setUserId((prev) => prev + 1)}>Next User</button>
      {loading && <p>Loading data...</p>}
      {error && <p style={{ color: "red" }}>Error: {error}</p>}
      {data && <p>Post Title: {data}</p>}
    </div>
  );
};

export default DataFetcher;
```

## 3. `useRef`

Provides a way to access DOM elements directly or persist mutable values across renders without causing re-renders.

```typescript
import React, { useRef, useEffect, useState } from "react";

const RefExample: React.FC = () => {
  const inputRef = useRef<HTMLInputElement>(null);
  const countRef = useRef<number>(0); // Mutable value that won't trigger re-render
  const [renderCount, setRenderCount] = useState<number>(0);

  useEffect(() => {
    // Focus the input field when the component mounts
    if (inputRef.current) {
      inputRef.current.focus();
    }
  }, []);

  const handleButtonClick = () => {
    if (inputRef.current) {
      alert(`Input value: ${inputRef.current.value}`);
    }
  };

  const incrementCountRef = () => {
    countRef.current = countRef.current + 1;
    console.log("countRef.current:", countRef.current);
    // This won't cause a re-render, so the UI won't update automatically
  };

  const triggerReRender = () => {
    setRenderCount((prev) => prev + 1); // This will cause a re-render
  };

  return (
    <div>
      <h2>useRef Example</h2>
      <input
        type="text"
        ref={inputRef}
        placeholder="Focus me on mount"
        className="border p-2 mr-2"
      />
      <button
        onClick={handleButtonClick}
        className="bg-blue-500 text-white px-3 py-1 rounded"
      >
        Get Input Value
      </button>

      <p className="mt-4">
        Mutable value (countRef): {countRef.current} (updates but doesn't
        re-render)
      </p>
      <button
        onClick={incrementCountRef}
        className="bg-green-500 text-white px-3 py-1 rounded mr-2"
      >
        Increment Count Ref
      </button>
      <button
        onClick={triggerReRender}
        className="bg-purple-500 text-white px-3 py-1 rounded"
      >
        Trigger Re-render ({renderCount})
      </button>
    </div>
  );
};

export default RefExample;
```

## 4. `useReducer`

An alternative to `useState` for more complex state logic, especially when state transitions depend on the previous state or involve multiple sub-values.

```typescript
import React, { useReducer } from "react";

// 1. Define the State Type
interface TodoState {
  todos: { id: number; text: string; completed: boolean }[];
  nextId: number;
}

// 2. Define Action Types
type TodoAction =
  | { type: "ADD_TODO"; text: string }
  | { type: "TOGGLE_TODO"; id: number }
  | { type: "DELETE_TODO"; id: number };

// 3. Create the Reducer Function
const todoReducer = (state: TodoState, action: TodoAction): TodoState => {
  switch (action.type) {
    case "ADD_TODO":
      return {
        ...state,
        todos: [
          ...state.todos,
          { id: state.nextId, text: action.text, completed: false },
        ],
        nextId: state.nextId + 1,
      };
    case "TOGGLE_TODO":
      return {
        ...state,
        todos: state.todos.map((todo) =>
          todo.id === action.id ? { ...todo, completed: !todo.completed } : todo
        ),
      };
    case "DELETE_TODO":
      return {
        ...state,
        todos: state.todos.filter((todo) => todo.id !== action.id),
      };
    default:
      return state;
  }
};

const TodoApp: React.FC = () => {
  // 4. Initialize useReducer
  const [state, dispatch] = useReducer(todoReducer, {
    todos: [],
    nextId: 1,
  });

  const handleAddTodo = (text: string) => {
    dispatch({ type: "ADD_TODO", text });
  };

  const handleToggleTodo = (id: number) => {
    dispatch({ type: "TOGGLE_TODO", id });
  };

  const handleDeleteTodo = (id: number) => {
    dispatch({ type: "DELETE_TODO", id });
  };

  return (
    <div className="p-4">
      <h2 className="text-2xl font-bold mb-4">Todo List (useReducer)</h2>
      <div className="flex mb-4">
        <input
          type="text"
          placeholder="Add a new todo"
          onKeyDown={(e) => {
            if (e.key === "Enter" && e.currentTarget.value.trim()) {
              handleAddTodo(e.currentTarget.value.trim());
              e.currentTarget.value = "";
            }
          }}
          className="border p-2 flex-grow mr-2"
        />
        <button
          onClick={() => {
            const input = document.querySelector("input") as HTMLInputElement;
            if (input.value.trim()) {
              handleAddTodo(input.value.trim());
              input.value = "";
            }
          }}
          className="bg-blue-500 text-white px-4 py-2 rounded"
        >
          Add Todo
        </button>
      </div>
      <ul className="space-y-2">
        {state.todos.map((todo) => (
          <li
            key={todo.id}
            className="flex items-center justify-between p-2 border rounded"
          >
            <span
              style={{
                textDecoration: todo.completed ? "line-through" : "none",
              }}
              onClick={() => handleToggleTodo(todo.id)}
              className="cursor-pointer flex-grow"
            >
              {todo.text}
            </span>
            <button
              onClick={() => handleDeleteTodo(todo.id)}
              className="bg-red-500 text-white px-2 py-1 rounded ml-4"
            >
              Delete
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default TodoApp;
```

## 5. `useMemo`

Memoizes a value, preventing expensive re-calculations on every render unless its dependencies change.

```typescript
import React, { useState, useMemo } from "react";

const ExpensiveCalculation: React.FC = () => {
  const [count, setCount] = useState<number>(0);
  const [multiplier, setMultiplier] = useState<number>(2);
  const [text, setText] = useState<string>("");

  // Simulate an expensive calculation
  const calculateExpensiveValue = (num: number, mult: number) => {
    console.log("Calculating expensive value...");
    // Imagine a loop or complex operation here
    let result = 0;
    for (let i = 0; i < 100000000; i++) {
      result += num * mult;
    }
    return result;
  };

  // useMemo will only re-run calculateExpensiveValue if count or multiplier changes
  const memoizedExpensiveValue = useMemo(() => {
    return calculateExpensiveValue(count, multiplier);
  }, [count, multiplier]); // Dependencies

  return (
    <div className="p-4">
      <h2 className="text-2xl font-bold mb-4">useMemo Example</h2>

      <div className="mb-4">
        <p>Count: {count}</p>
        <button
          onClick={() => setCount((prev) => prev + 1)}
          className="bg-blue-500 text-white px-3 py-1 rounded mr-2"
        >
          Increment Count
        </button>
      </div>

      <div className="mb-4">
        <p>Multiplier: {multiplier}</p>
        <button
          onClick={() => setMultiplier((prev) => prev + 1)}
          className="bg-green-500 text-white px-3 py-1 rounded mr-2"
        >
          Increment Multiplier
        </button>
      </div>

      <div className="mb-4">
        <p>Expensive Value (memoized): {memoizedExpensiveValue}</p>
        <p className="text-sm text-gray-600">
          (This value only re-calculates when Count or Multiplier changes)
        </p>
      </div>

      <div>
        <label
          htmlFor="textInput"
          className="block text-sm font-medium text-gray-700"
        >
          Type something (doesn't affect expensive calculation):
        </label>
        <input
          type="text"
          id="textInput"
          value={text}
          onChange={(e) => setText(e.target.value)}
          className="border p-2 mt-1 w-full"
        />
        <p>You typed: {text}</p>
      </div>
    </div>
  );
};

export default ExpensiveCalculation;
```
