# Common React UI Patterns Cheat Sheet

This document provides quick reference examples for common React UI patterns.

## 1. Button Component

A simple, reusable button component.

````typescript
import React from "react";

interface ButtonProps {
  onClick: () => void;
  children: React.ReactNode;
  variant?: "primary" | "secondary" | "danger";
  disabled?: boolean;
}

const Button: React.FC<ButtonProps> = ({
  onClick,
  children,
  variant = "primary",
  disabled = false,
}) => {
  const baseStyle = "px-4 py-2 rounded font-semibold";
  let variantStyle = "";

  switch (variant) {
    case "primary":
      variantStyle = "bg-blue-500 text-white hover:bg-blue-600";
      break;
    case "secondary":
      variantStyle = "bg-gray-300 text-gray-800 hover:bg-gray-400";
      break;
    case "danger":
      variantStyle = "bg-red-500 text-white hover:bg-red-600";
      break;
  }

  const disabledStyle = disabled ? "opacity-50 cursor-not-allowed" : "";

  return (
    <button
      className={`${baseStyle} ${variantStyle} ${disabledStyle}`}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
};

export default Button;

// Usage Example:
// import Button from './Button';
//
// const MyPage = () => {
//   const handleClick = () => alert('Button clicked!');
//   return (
//     <div>
//       <Button onClick={handleClick} variant="primary">Primary Button</Button>
//       <Button onClick={handleClick} variant="secondary">Secondary Button</Button>
//       <Button onClick={handleClick} variant="danger" disabled>Disabled Button</Button>
//     </div>
//   );
// };

### Explanation of `children` Prop

The `children` prop is a special prop in React that allows you to pass components or elements as data to other components by nesting them inside the opening and closing tags.

**Significance:**

1.  **Content Flexibility**: It makes components highly reusable and flexible. Instead of defining a specific `text` prop for the button's label, `children` allows you to pass any valid React node (text, other components, HTML elements) directly between the `<Button>` and `</Button>` tags.
    *   **Example**:
        ```jsx
        <Button onClick={handleClick}>Click Me</Button> // 'Click Me' is the children
        <Button onClick={handleIconClick}><img src="icon.png" alt="icon" /></Button> // An image is the children
        ```
2.  **Component Composition**: It's a core concept for component composition, enabling you to build complex UIs by combining simpler components. The parent component (e.g., `Button`) doesn't need to know the exact content it will render; it just renders whatever is passed as `children`.
3.  **Standard Practice**: Using `children` for content is the idiomatic React way for components that act as wrappers or containers for other content.

In the `Button` component, `children: React.ReactNode;` in the `ButtonProps` interface types the `children` prop, indicating it can be any renderable React content (strings, numbers, elements, arrays of elements, or fragments). The `{children}` inside the `<button>` element then renders this content.
````

## 2. Form with Validation (Controlled Component)

An example of a controlled form component with basic client-side validation.

```typescript
import React, { useState, FormEvent, ChangeEvent } from "react";

interface FormData {
  username: string;
  email: string;
}

interface FormErrors {
  username?: string;
  email?: string;
}

const MyForm: React.FC = () => {
  const [formData, setFormData] = useState<FormData>({
    username: "",
    email: "",
  });
  const [errors, setErrors] = useState<FormErrors>({});
  const [isSubmitted, setIsSubmitted] = useState<boolean>(false);

  const validate = (): FormErrors => {
    const newErrors: FormErrors = {};
    if (!formData.username) {
      newErrors.username = "Username is required";
    } else if (formData.username.length < 3) {
      newErrors.username = "Username must be at least 3 characters";
    }
    if (!formData.email) {
      newErrors.email = "Email is required";
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = "Email address is invalid";
    }
    return newErrors;
  };

  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData((prev) => ({ ...prev, [name]: value }));
    // Clear error for the field as user types
    if (errors[name]) {
      setErrors((prev) => ({ ...prev, [name]: undefined }));
    }
  };

  const handleSubmit = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const validationErrors = validate();
    setErrors(validationErrors);
    setIsSubmitted(true);

    if (Object.keys(validationErrors).length === 0) {
      alert("Form submitted successfully: " + JSON.stringify(formData));
      // Here you would typically send data to a backend
    } else {
      console.log("Form has errors:", validationErrors);
    }
  };

  return (
    <form
      onSubmit={handleSubmit}
      className="space-y-4 p-4 border rounded shadow"
    >
      <div>
        <label
          htmlFor="username"
          className="block text-sm font-medium text-gray-700"
        >
          Username:
        </label>
        <input
          type="text"
          id="username"
          name="username"
          value={formData.username}
          onChange={handleChange}
          className="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
        />
        {errors.username && (
          <p className="text-red-500 text-xs mt-1">{errors.username}</p>
        )}
      </div>
      <div>
        <label
          htmlFor="email"
          className="block text-sm font-medium text-gray-700"
        >
          Email:
        </label>
        <input
          type="email"
          id="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          className="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
        />
        {errors.email && (
          <p className="text-red-500 text-xs mt-1">{errors.email}</p>
        )}
      </div>
      <button
        type="submit"
        className="bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600"
      >
        Submit
      </button>
    </form>
  );
};

export default MyForm;
```

## 3. Modal Component

A basic modal component that can be toggled. Uses `ReactDOM.createPortal` for better accessibility and layering.

```typescript
import React, { useEffect, useRef } from "react";
import ReactDOM from "react-dom";

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  children: React.ReactNode;
  title?: string;
}

const Modal: React.FC<ModalProps> = ({
  isOpen,
  onClose,
  children,
  title = "Modal Title",
}) => {
  const modalRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const handleEscape = (event: KeyboardEvent) => {
      if (event.key === "Escape") {
        onClose();
      }
    };

    if (isOpen) {
      document.addEventListener("keydown", handleEscape);
      document.body.style.overflow = "hidden"; // Prevent scrolling background
    } else {
      document.body.style.overflow = "unset";
    }

    return () => {
      document.removeEventListener("keydown", handleEscape);
      document.body.style.overflow = "unset";
    };
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return ReactDOM.createPortal(
    <div className="fixed inset-0 bg-gray-600 bg-opacity-50 flex justify-center items-center z-50">
      <div
        ref={modalRef}
        className="bg-white rounded-lg shadow-xl p-6 w-11/12 md:w-1/2 lg:w-1/3 relative"
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
      >
        <h2 id="modal-title" className="text-xl font-bold mb-4">
          {title}
        </h2>
        <button
          onClick={onClose}
          className="absolute top-3 right-3 text-gray-500 hover:text-gray-700 text-2xl"
          aria-label="Close modal"
        >
          &times;
        </button>
        <div>{children}</div>
      </div>
    </div>,
    document.body // Portal to the body element
  );
};

export default Modal;

// Usage Example:
// import React, { useState } from 'react';
// import Modal from './Modal';
// import Button from './Button'; // Assuming you have a Button component
//
// const MyPage = () => {
//   const [isModalOpen, setIsModalOpen] = useState(false);
//
//   return (
//     <div>
//       <Button onClick={() => setIsModalOpen(true)}>Open Modal</Button>
//
//       <Modal isOpen={isModalOpen} onClose={() => setIsModalOpen(false)} title="Welcome to the Modal">
//         <p>This is the content of the modal.</p>
//         <Button onClick={() => setIsModalOpen(false)} variant="secondary">Close</Button>
//       </Modal>
//     </div>
//   );
// };
```

## 4. Select Input Component

A controlled select input component.

```typescript
import React, { useState, ChangeEvent } from "react";

interface SelectOption {
  value: string;
  label: string;
}

interface SelectInputProps {
  options: SelectOption[];
  label: string;
  value: string;
  onChange: (value: string) => void;
  name: string;
  id: string;
}

const SelectInput: React.FC<SelectInputProps> = ({
  options,
  label,
  value,
  onChange,
  name,
  id,
}) => {
  const handleChange = (e: ChangeEvent<HTMLSelectElement>) => {
    onChange(e.target.value);
  };

  return (
    <div className="mb-4">
      <label
        htmlFor={id}
        className="block text-sm font-medium text-gray-700 mb-1"
      >
        {label}
      </label>
      <select
        id={id}
        name={name}
        value={value}
        onChange={handleChange}
        className="mt-1 block w-full pl-3 pr-10 py-2 text-base border-gray-300 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm rounded-md"
      >
        {options.map((option) => (
          <option key={option.value} value={option.value}>
            {option.label}
          </option>
        ))}
      </select>
    </div>
  );
};

export default SelectInput;

// Usage Example:
// import React, { useState } from 'react';
// import SelectInput from './SelectInput';
//
// const MyPage = () => {
//   const [selectedFruit, setSelectedFruit] = useState('apple');
//
//   const fruitOptions = [
//     { value: 'apple', label: 'Apple' },
//     { value: 'banana', label: 'Banana' },
//     { value: 'orange', label: 'Orange' },
//   ];
//
//   return (
//     <div className="p-4">
//       <SelectInput
//         id="fruit-select"
//         name="fruit"
//         label="Choose a Fruit:"
//         options={fruitOptions}
//         value={selectedFruit}
//         onChange={setSelectedFruit}
//       />
//       <p>Selected: {selectedFruit}</p>
//     </div>
//   );
// };
```

## 5. Simple Form

```typescript
import React, { useState } from "react";

export const AddTaskForm = ({ addNewTask }: TaskFormProps) => {
  const [inputValue, setInputValue] = useState<string>("");

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!inputValue.trim()) return;
    const trimmedInput = inputValue.trim();
    addNewTask(trimmedInput);
    setInputValue("");
  };

  const handleChange = (e) => {
    setInputValue(e.target.value);
  };

  return (
    <>
      <h3>Add Task</h3>
      <form onSubmit={handleSubmit}>
        <input type="text" onChange={handleChange} value={inputValue} />
        <button type="submit">Submit</button>
      </form>
    </>
  );
};
```

## 6. Toggle Function

```typescript
const toggleCompleted = (id: string) => {
  setTasks((prev) =>
    prev.map((t) => (t.id === id ? { ...t, completed: !t.completed } : t))
  );
};
```

## 6. Creat Function

```typescript
const addNewTask = (title) => {
  setTasks((prev) => [
    ...prev,
    { id: Date.now().toString(), title: title, completed: false },
  ]);
};
```

## 6. Delete Function

```typescript
const deleteItem = (id: string) => {
  const updatedItems = tasks.filter((task) => task.id !== id);
  setTasks(updatedItems);
};
```
