# Common React Patterns Cheat Sheet

This document provides quick reference examples for common React patterns, focusing on reusability, performance, and maintainability.

## 1. Filterable List (Basic)

A simple component to filter a list of items without memoization.

```typescript
import React, { useState, ChangeEvent } from "react";

interface Item {
  id: number;
  name: string;
  category: string;
}

interface FilterableListProps {
  items: Item[];
}

const FilterableList: React.FC<FilterableListProps> = ({ items }) => {
  const [searchTerm, setSearchTerm] = useState<string>("");
  const [selectedCategory, setSelectedCategory] = useState<string>("All");

  const handleSearchChange = (e: ChangeEvent<HTMLInputElement>) => {
    setSearchTerm(e.target.value);
  };

  const handleCategoryChange = (e: ChangeEvent<HTMLSelectElement>) => {
    setSelectedCategory(e.target.value);
  };

  // This filtering logic runs on every render
  const filteredItems = items.filter((item) => {
    const matchesSearch = item.name
      .toLowerCase()
      .includes(searchTerm.toLowerCase());
    const matchesCategory =
      selectedCategory === "All" || item.category === selectedCategory;
    return matchesSearch && matchesCategory;
  });

  const categories = ["All", ...new Set(items.map((item) => item.category))];

  return (
    <div className="p-4 border rounded shadow">
      <h2 className="text-2xl font-bold mb-4">Filterable List (Basic)</h2>
      <div className="mb-4 flex space-x-4">
        <input
          type="text"
          placeholder="Search by name..."
          value={searchTerm}
          onChange={handleSearchChange}
          className="border p-2 flex-grow rounded"
        />
        <select
          value={selectedCategory}
          onChange={handleCategoryChange}
          className="border p-2 rounded"
        >
          {categories.map((category) => (
            <option key={category} value={category}>
              {category}
            </option>
          ))}
        </select>
      </div>
      <ul className="space-y-2">
        {filteredItems.map((item) => (
          <li key={item.id} className="p-2 border rounded bg-gray-50">
            {item.name} ({item.category})
          </li>
        ))}
      </ul>
    </div>
  );
};

export default FilterableList;

// Usage Example:
// const myItems = [
//   { id: 1, name: 'Apple', category: 'Fruit' },
//   { id: 2, name: 'Banana', category: 'Fruit' },
//   { id: 3, name: 'Carrot', category: 'Vegetable' },
//   { id: 4, name: 'Broccoli', category: 'Vegetable' },
//   { id: 5, name: 'Milk', category: 'Dairy' },
// ];
// <FilterableList items={myItems} />
```

## 2. Filterable List with Memoization (`useMemo`)

Optimizing the filter logic using `useMemo` to prevent unnecessary re-calculations.

```typescript
import React, { useState, ChangeEvent, useMemo } from "react";

interface Item {
  id: number;
  name: string;
  category: string;
}

interface FilterableListMemoizedProps {
  items: Item[];
}

const FilterableListMemoized: React.FC<FilterableListMemoizedProps> = ({
  items,
}) => {
  const [searchTerm, setSearchTerm] = useState<string>("");
  const [selectedCategory, setSelectedCategory] = useState<string>("All");
  const [extraInput, setExtraInput] = useState<string>(""); // An unrelated state to demonstrate memoization

  const handleSearchChange = (e: ChangeEvent<HTMLInputElement>) => {
    setSearchTerm(e.target.value);
  };

  const handleCategoryChange = (e: ChangeEvent<HTMLSelectElement>) => {
    setSelectedCategory(e.target.value);
  };

  // Memoize the filtered items. This function will only re-run if 'items', 'searchTerm',
  // or 'selectedCategory' changes. Changes to 'extraInput' will NOT trigger re-calculation.
  const filteredItems = useMemo(() => {
    console.log("Recalculating filtered items..."); // Will only log when dependencies change
    return items.filter((item) => {
      const matchesSearch = item.name
        .toLowerCase()
        .includes(searchTerm.toLowerCase());
      const matchesCategory =
        selectedCategory === "All" || item.category === selectedCategory;
      return matchesSearch && matchesCategory;
    });
  }, [items, searchTerm, selectedCategory]); // Dependencies for memoization

  const categories = useMemo(() => {
    return ["All", ...new Set(items.map((item) => item.category))];
  }, [items]); // Categories only re-calculate if 'items' changes

  return (
    <div className="p-4 border rounded shadow mt-8">
      <h2 className="text-2xl font-bold mb-4">Filterable List (Memoized)</h2>
      <div className="mb-4 flex space-x-4">
        <input
          type="text"
          placeholder="Search by name..."
          value={searchTerm}
          onChange={handleSearchChange}
          className="border p-2 flex-grow rounded"
        />
        <select
          value={selectedCategory}
          onChange={handleCategoryChange}
          className="border p-2 rounded"
        >
          {categories.map((category) => (
            <option key={category} value={category}>
              {category}
            </option>
          ))}
        </select>
      </div>
      <div className="mb-4">
        <input
          type="text"
          placeholder="Type something unrelated (won't trigger filter recalculation)"
          value={extraInput}
          onChange={(e) => setExtraInput(e.target.value)}
          className="border p-2 w-full rounded"
        />
        <p className="text-sm text-gray-600 mt-1">Extra input: {extraInput}</p>
      </div>
      <ul className="space-y-2">
        {filteredItems.map((item) => (
          <li key={item.id} className="p-2 border rounded bg-gray-50">
            {item.name} ({item.category})
          </li>
        ))}
      </ul>
    </div>
  );
};

export default FilterableListMemoized;

// Usage Example:
// const myItems = [
//   { id: 1, name: 'Apple', category: 'Fruit' },
//   { id: 2, name: 'Banana', category: 'Fruit' },
//   { id: 3, name: 'Carrot', category: 'Vegetable' },
//   { id: 4, name: 'Broccoli', category: 'Vegetable' },
//   { id: 5, name: 'Milk', category: 'Dairy' },
// ];
// <FilterableListMemoized items={myItems} />
```

## 3. Controlled vs. Uncontrolled Components

Understanding the difference and when to use each.

### Controlled Component (Recommended)

State is managed by React. Form elements' values are driven by React state.

```typescript
import React, { useState, ChangeEvent } from "react";

const ControlledInput: React.FC = () => {
  const [value, setValue] = useState<string>("");

  const handleChange = (event: ChangeEvent<HTMLInputElement>) => {
    setValue(event.target.value);
  };

  return (
    <div className="p-4 border rounded shadow mt-8">
      <h2 className="text-2xl font-bold mb-4">Controlled Component</h2>
      <input
        type="text"
        value={value}
        onChange={handleChange}
        placeholder="Type here..."
        className="border p-2 rounded w-full"
      />
      <p className="mt-2">Current Value: {value}</p>
    </div>
  );
};

export default ControlledInput;
```

### Uncontrolled Component (Less Common, for simple cases)

DOM manages its own state. Use a `ref` to get its value when needed.

```typescript
import React, { useRef, FormEvent } from "react";

const UncontrolledInput: React.FC = () => {
  const inputRef = useRef<HTMLInputElement>(null);

  const handleSubmit = (event: FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    alert(`Input value: ${inputRef.current?.value}`);
  };

  return (
    <div className="p-4 border rounded shadow mt-4">
      <h2 className="text-2xl font-bold mb-4">Uncontrolled Component</h2>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          ref={inputRef}
          defaultValue="Initial Value" // Use defaultValue for uncontrolled inputs
          className="border p-2 rounded w-full mb-2"
        />
        <button
          type="submit"
          className="bg-blue-500 text-white px-4 py-2 rounded"
        >
          Submit
        </button>
      </form>
    </div>
  );
};

export default UncontrolledInput;
```

## 4. Higher-Order Components (HOCs)

A pattern for reusing component logic. (Note: Hooks are generally preferred for new code).

```typescript
import React, { useState, ComponentType } from "react";

interface WithLoadingProps {
  isLoading: boolean;
}

// HOC: A function that takes a component and returns a new component
const withLoading = <P extends object>(
  WrappedComponent: ComponentType<P & WithLoadingProps>
) => {
  const ComponentWithLoading: React.FC<P> = (props) => {
    const [isLoading, setIsLoading] = useState<boolean>(true);

    // Simulate data fetching
    React.useEffect(() => {
      const timer = setTimeout(() => {
        setIsLoading(false);
      }, 2000);
      return () => clearTimeout(timer);
    }, []);

    return <WrappedComponent {...(props as P)} isLoading={isLoading} />;
  };
  return ComponentWithLoading;
};

// A component that will be wrapped by the HOC
interface MyDataDisplayProps {
  data: string;
  isLoading: boolean; // This prop will be provided by the HOC
}

const MyDataDisplay: React.FC<MyDataDisplayProps> = ({ data, isLoading }) => {
  if (isLoading) {
    return <div>Loading data...</div>;
  }
  return <div>Data: {data}</div>;
};

// Create the enhanced component
const EnhancedDataDisplay = withLoading(MyDataDisplay);

// Usage Example:
// const App = () => {
//   return (
//     <div className="p-4">
//       <h2 className="text-2xl font-bold mb-4">HOC Example</h2>
//       <EnhancedDataDisplay data="Some important information" />
//     </div>
//   );
// };
// export default App;
```

## 5. Render Props

Another pattern for sharing code between React components using a prop whose value is a function.

```typescript
import React, { useState, ReactNode } from "react";

interface MouseTrackerProps {
  render: (state: { x: number; y: number }) => ReactNode;
}

const MouseTracker: React.FC<MouseTrackerProps> = ({ render }) => {
  const [mousePosition, setMousePosition] = useState<{ x: number; y: number }>({
    x: 0,
    y: 0,
  });

  const handleMouseMove = (event: React.MouseEvent<HTMLDivElement>) => {
    setMousePosition({
      x: event.clientX,
      y: event.clientY,
    });
  };

  return (
    <div
      style={{
        height: "200px",
        border: "1px solid gray",
        position: "relative",
      }}
      onMouseMove={handleMouseMove}
      className="p-4"
    >
      <h3>Move your mouse over this area!</h3>
      {render(mousePosition)}
    </div>
  );
};

// Usage Example:
// const App = () => {
//   return (
//     <div className="p-4">
//       <h2 className="text-2xl font-bold mb-4">Render Props Example</h2>
//       <MouseTracker
//         render={({ x, y }) => (
//           <p>The mouse position is ({x}, {y})</p>
//         )}
//       />
//       <MouseTracker
//         render={({ x, y }) => (
//           <h1 style={{ position: 'absolute', left: x, top: y }}>
//             &#128044;
//           </h1>
//         )}
//       />
//     </div>
//   );
// };
// export default App;
```

## 6. Custom Hooks

A pattern for extracting and reusing stateful logic from components.

```typescript
import React, { useState, useEffect } from "react";

// Custom Hook: useDebounce
// Delays updating a value until a certain time has passed since the last change.
// Useful for search inputs, preventing excessive API calls.
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}

const SearchInputWithDebounce: React.FC = () => {
  const [searchTerm, setSearchTerm] = useState<string>("");
  const debouncedSearchTerm = useDebounce(searchTerm, 500); // Debounce for 500ms

  useEffect(() => {
    if (debouncedSearchTerm) {
      console.log("Fetching data for:", debouncedSearchTerm);
      // In a real app, you'd make an API call here
    }
  }, [debouncedSearchTerm]);

  return (
    <div className="p-4 border rounded shadow mt-8">
      <h2 className="text-2xl font-bold mb-4">Custom Hook: useDebounce</h2>
      <input
        type="text"
        placeholder="Search..."
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        className="border p-2 rounded w-full"
      />
      <p className="mt-2">
        Actual search term (debounced): {debouncedSearchTerm}
      </p>
      <p className="text-sm text-gray-600">
        (Check console for "Fetching data" logs, they are delayed)
      </p>
    </div>
  );
};

export default SearchInputWithDebounce;

// Usage Example:
// <SearchInputWithDebounce />
```

## 7. Error Boundaries

A React component that catches JavaScript errors anywhere in its child component tree, logs those errors, and displays a fallback UI instead of the component tree crashing.

```typescript
import React, { Component, ErrorInfo, ReactNode } from "react";

interface ErrorBoundaryProps {
  children: ReactNode;
  fallback?: ReactNode;
}

interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

class ErrorBoundary extends Component<ErrorBoundaryProps, ErrorBoundaryState> {
  public state: ErrorBoundaryState = {
    hasError: false,
    error: null,
  };

  // This static method is called after an error has been thrown by a descendant component.
  // It receives the error that was thrown as a parameter and should return a value to update state.
  public static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    // Update state so the next render will show the fallback UI.
    return { hasError: true, error };
  }

  // This method is called after an error has been thrown by a descendant component.
  // It receives two parameters: error and info.
  public componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error("Uncaught error:", error, errorInfo);
    // You can also log the error to an error reporting service here
    // logErrorToMyService(error, errorInfo);
  }

  public render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return (
        this.props.fallback || (
          <div className="p-4 bg-red-100 border border-red-400 text-red-700 rounded relative">
            <h2 className="font-bold">Something went wrong.</h2>
            <p>
              {this.state.error?.message || "An unexpected error occurred."}
            </p>
            <p className="text-sm mt-2">Please try refreshing the page.</p>
          </div>
        )
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;

// Example of a component that might throw an error
const BuggyComponent: React.FC<{ shouldThrow: boolean }> = ({
  shouldThrow,
}) => {
  if (shouldThrow) {
    throw new Error("I crashed!");
  }
  return <p>This component is fine.</p>;
};

// Usage Example:
// import React, { useState } from 'react';
// import ErrorBoundary from './ErrorBoundary';
// import Button from './Button'; // Assuming you have a Button component
//
// const App = () => {
//   const [throwError, setThrowError] = useState(false);
//
//   return (
//     <div className="p-4">
//       <h2 className="text-2xl font-bold mb-4">Error Boundary Example</h2>
//       <Button onClick={() => setThrowError(true)} variant="danger">
//         Trigger Error
//       </Button>
//       <div className="mt-4 border p-4 rounded">
//         <ErrorBoundary fallback={<p className="text-purple-700">Custom fallback UI for error!</p>}>
//           <BuggyComponent shouldThrow={throwError} />
//         </ErrorBoundary>
//       </div>
//       <p className="mt-4">This content is outside the ErrorBoundary and will not crash.</p>
//     </div>
//   );
// };
// export default App;
```

## 8. Compound Components

A pattern where components work together to share implicit state and logic, but give the user full control over rendering.

```typescript
import React, { useState, createContext, useContext, ReactNode } from "react";

// 1. Create Context for shared state
interface TabsContextType {
  activeTab: string;
  setActiveTab: (tabId: string) => void;
}
const TabsContext = createContext<TabsContextType | undefined>(undefined);

// Custom hook to use the TabsContext
const useTabs = () => {
  const context = useContext(TabsContext);
  if (context === undefined) {
    throw new Error("useTabs must be used within a Tabs component");
  }
  return context;
};

// 2. Parent Component (Tabs) - manages state and provides context
interface TabsProps {
  children: ReactNode;
  defaultActiveTab: string;
}

const Tabs: React.FC<TabsProps> = ({ children, defaultActiveTab }) => {
  const [activeTab, setActiveTab] = useState<string>(defaultActiveTab);

  const contextValue: TabsContextType = { activeTab, setActiveTab };

  return (
    <TabsContext.Provider value={contextValue}>
      <div className="border rounded p-4">{children}</div>
    </TabsContext.Provider>
  );
};

// 3. Child Component (TabList) - renders tab buttons
interface TabListProps {
  children: ReactNode;
}

const TabList: React.FC<TabListProps> = ({ children }) => {
  return <div className="flex border-b mb-4">{children}</div>;
};

// 4. Child Component (TabButton) - individual tab button
interface TabButtonProps {
  tabId: string;
  children: ReactNode;
}

const TabButton: React.FC<TabButtonProps> = ({ tabId, children }) => {
  const { activeTab, setActiveTab } = useTabs();
  const isActive = activeTab === tabId;

  const buttonStyle = `px-4 py-2 -mb-px border-b-2 ${
    isActive
      ? "border-blue-500 text-blue-600 font-semibold"
      : "border-transparent text-gray-600 hover:text-gray-800"
  }`;

  return (
    <button onClick={() => setActiveTab(tabId)} className={buttonStyle}>
      {children}
    </button>
  );
};

// 5. Child Component (TabPanel) - renders content for active tab
interface TabPanelProps {
  tabId: string;
  children: ReactNode;
}

const TabPanel: React.FC<TabPanelProps> = ({ tabId, children }) => {
  const { activeTab } = useTabs();
  return activeTab === tabId ? <div>{children}</div> : null;
};

// Export all components for easy access
export { Tabs, TabList, TabButton, TabPanel };

// Usage Example:
// import { Tabs, TabList, TabButton, TabPanel } from './Tabs'; // Assuming these are in Tabs.tsx
//
// const App = () => {
//   return (
//     <div className="p-4">
//       <h2 className="text-2xl font-bold mb-4">Compound Components Example</h2>
//       <Tabs defaultActiveTab="tab1">
//         <TabList>
//           <TabButton tabId="tab1">First Tab</TabButton>
//           <TabButton tabId="tab2">Second Tab</TabButton>
//           <TabButton tabId="tab3">Third Tab</TabButton>
//         </TabList>
//
//         <TabPanel tabId="tab1">
//           <p>Content for the first tab.</p>
//         </TabPanel>
//         <TabPanel tabId="tab2">
//           <p>Content for the second tab. This can be anything!</p>
//         </TabPanel>
//         <TabPanel tabId="tab3">
//           <ul>
//             <li>Item 1</li>
//             <li>Item 2</li>
//           </ul>
//         </TabPanel>
//       </Tabs>
//     </div>
//   );
// };
// export default App;
```

## 9. Data Fetching Pattern (`useEffect` and `useState`)

A common pattern for fetching asynchronous data from an API and managing loading and error states.

```typescript
import React, { useState, useEffect } from "react";

interface Post {
  id: number;
  title: string;
  body: string;
}

const DataFetcherComponent: React.FC = () => {
  const [posts, setPosts] = useState<Post[]>([]);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchPosts = async () => {
      try {
        setLoading(true);
        setError(null);
        const response = await fetch(
          "https://jsonplaceholder.typicode.com/posts?_limit=5"
        );
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data: Post[] = await response.json();
        setPosts(data);
      } catch (err: any) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchPosts(); // Call the async function inside useEffect

    // Optional cleanup function
    return () => {
      // Any cleanup logic here, e.g., aborting fetch requests
    };
  }, []); // Empty dependency array means this effect runs once on mount

  if (loading) {
    return <div className="p-4">Loading posts...</div>;
  }

  if (error) {
    return <div className="p-4 text-red-600">Error: {error}</div>;
  }

  return (
    <div className="p-4 border rounded shadow mt-8">
      <h2 className="text-2xl font-bold mb-4">Data Fetching Example</h2>
      <ul className="space-y-4">
        {posts.map((post) => (
          <li key={post.id} className="p-3 border rounded bg-blue-50">
            <h3 className="text-lg font-semibold">{post.title}</h3>
            <p className="text-gray-700">{post.body.substring(0, 100)}...</p>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default DataFetcherComponent;

// Usage Example:
// <DataFetcherComponent />
```
