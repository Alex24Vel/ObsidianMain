Source: https://react.dev/learn/build-a-react-app-from-scratch

Date: 10.05.2025 14:30

# 1. Installing React TypeScript + SWC using [Vite](https://react.dev/learn/build-a-react-app-from-scratch#vite)
```Bash
%% Navigate to client project folder first, a react subfolder will be created there %%
cd "Path to client project"

npm create vite@latest --template react .

> npx
> create-vite react .

│
◆  Select a framework:
│  ○ Vanilla
│  ○ Vue
│  ● React
│  ○ Preact
│  ○ Lit
│  ○ Svelte
│  ○ Solid
│  ○ Qwik
│  ○ Angular
│  ○ Marko
│  ○ Others
└

◆  Select a variant:
│  ○ TypeScript
│  ● TypeScript + SWC
│  ○ JavaScript
│  ○ JavaScript + SWC
│  ○ React Router v7 ↗
│  ○ TanStack Router ↗
│  ○ RedwoodSDK ↗
└

◇  Select a framework:
│  React
│
◇  Select a variant:
│  TypeScript + SWC
│
◇  Scaffolding project in "ProjectPath\ProjectName\react..."
│
└  Done. Now run:

  cd react
  npm install
  npm run dev
```

# 2. Installing React Router

```Bash
npm install react-router-dom
```

# 3. Setting up React Router

1. **Set up React Router in your application:**
Now you need to integrate React Router into your application. The typical place to do this is in your main entry file, usually `src/main.tsx` (since you chose TypeScript).

Open `src/main.tsx` and modify it to use `BrowserRouter` (or `HashRouter` depending on your needs, but `BrowserRouter` is more common for standard web applications).

Here's an example of how you might modify `src/main.tsx`:

```typescript
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App.tsx';
import './index.css';
import { BrowserRouter } from 'react-router-dom'; // Import BrowserRouter

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <BrowserRouter> {/* Wrap your App with BrowserRouter */}
      <App />
    </BrowserRouter>
  </React.StrictMode>,
);
```

2. **Define your routes:**
Next, you'll define the different routes (paths) of your application and the components that should render for each path. This is typically done in your main `App.tsx` component or in a dedicated routing file.

Here's a simple example in `src/App.tsx`:
```typescript
import { useState } from 'react';
import reactLogo from './assets/react.svg';
import viteLogo from '/vite.svg';
import './App.css';
import { Routes, Route, Link } from 'react-router-dom'; // Import routing components

// Create some simple components for demonstration
const HomePage = () => <h1>Home Page</h1>;
const AboutPage = () => <h1>About Page</h1>;
const ContactPage = () => <h1>Contact Page</h1>;

function App() {
  const [count, setCount] = useState(0);

  return (
    <>
      <div>
        <a href="https://vitejs.dev" target="_blank">
          <img src={viteLogo} className="logo" alt="Vite logo" />
        </a>
        <a href="https://react.dev" target="_blank">
          <img src={reactLogo} className="logo react" alt="React logo" />
        </a>
      </div>
      <h1>Vite + React</h1>
      <div className="card">
        <button onClick={() => setCount((count) => count + 1)}>
          count is {count}
        </button>
        <p>
          Edit <code>src/App.tsx</code> and save to test HMR
        </p>
      </div>
      <p className="read-the-docs">
        Click on the Vite and React logos to learn more
      </p>

      {/* Navigation Links */}
      <nav>
        <ul>
          <li>
            <Link to="/">Home</Link>
          </li>
          <li>
            <Link to="/about">About</Link>
          </li>
          <li>
            <Link to="/contact">Contact</Link>
          </li>
        </ul>
      </nav>

      {/* Define your Routes */}
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/about" element={<AboutPage />} />
        <Route path="/contact" element={<ContactPage />} />
      </Routes>
    </>
  );
}

export default App;

```

In this example:
- We import Routes, Route, and Link from react-router-dom.
- Link is used to create navigation links that handle client-side routing without full page reloads.
- Routes is a container for defining multiple Route components.
- Route components define the path and the element (the component) to render when the URL matches that path.

Next Steps:
- Replace the placeholder components (HomePage, AboutPage, ContactPage) with your actual React components for different parts of your application.
- Organize your routes and components as your application grows. You might create a separate routes.tsx file for defining more complex routing structures.
- Explore more advanced React Router features like nested routes, route parameters, and programmatic navigation as needed.

# Why those options?

Okay, let's break down these choices for your React frontend with an ASP.NET Core backend.
You're absolutely right that if you're using React solely as a frontend client that talks to your ASP.NET Core API, you don't necessarily need a full-blown framework like Next.js. Next.js is excellent for server-side rendering, static site generation, and other features that are often geared towards having a full-stack application within the React ecosystem. Since your backend is already handled by ASP.NET Core, a standard React setup is perfectly suitable.
Now, regarding the npm create vite@latest options:
React Router vs. Standard JS/TS Variant:
This is the main decision point.

TypeScript / TypeScript + SWC / JavaScript / JavaScript + SWC: These options give you a basic React project setup using either JavaScript or TypeScript, and with either the standard build tools or SWC (Speedy Web Compiler). If you choose one of these, you will typically manually add a routing library like React Router or TanStack Router later if you need client-side routing (navigating between different "pages" in your single-page application without full page reloads). This gives you more control over how you set up routing, but it requires a few extra steps after the project is created.
React Router v7 ↗ (npm create react-router@latest): This option will create a new project specifically pre-configured with React Router. This is a convenience option if you know you'll be using React Router and want to get started with it already set up.
Which one to take?
Since you're building a single-page application (SPA) with React that will likely have different views or "pages" (e.g., a login page, a dashboard page, a details page), you will almost certainly need client-side routing.

If you are comfortable installing and setting up a library yourself after the project is created, choose one of the TypeScript or JavaScript options. This gives you the foundational project and you can add React Router (or another router) later.
If you prefer to have routing set up from the beginning with the most popular React routing library, choose the React Router v7 option. This is often the easiest way to get started with routing.
TypeScript vs. JavaScript:

TypeScript: Adds static typing to JavaScript. This means you define the expected types of variables, function arguments, and return values. This can catch many errors during development before you even run your code, leading to more robust and maintainable applications, especially as they grow in size and complexity.
JavaScript: The standard, dynamically typed language. It's quicker to get started with as you don't need to think about types initially, but larger projects can become harder to manage and refactor without the type safety that TypeScript provides.
Given your background with C# (which is a strongly typed language), you'll likely find TypeScript to be quite familiar and beneficial. I highly recommend starting with TypeScript for a new React project.
TypeScript + SWC vs. TypeScript:

SWC (Speedy Web Compiler): This is an alternative to the standard Babel compiler that's often used in JavaScript/TypeScript build pipelines. SWC is written in Rust and is generally much faster than Babel.
Difference in usage: From a developer's perspective, there is very little difference in how you write your React code using TypeScript with or without SWC. SWC just handles the process of compiling your TypeScript/JSX code into standard JavaScript more quickly. You won't have to learn anything new about writing React or TypeScript code itself. The difference is in the build performance (how quickly your code compiles and refreshes during development).
My Recommendation:
Given your setup and likely need for routing, and considering the benefits of type safety when coming from a C# background and the performance boost of SWC, I recommend selecting:

React Router v7 (for convenience in getting routing set up)
When prompted, choose TypeScript + SWC (for type safety and faster build times)
Alternatively, if you prefer to add React Router manually later:

Select TypeScript + SWC
Then, after the project is created, install react-router-dom: npm install react-router-dom and configure it in your project.
Both approaches are perfectly valid. The React Router v7 option in the create-vite prompt is just a shortcut to the second option.