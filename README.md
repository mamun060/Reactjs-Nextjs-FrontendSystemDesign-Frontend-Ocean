# Frontend React.js & Next.js Knowledge Brush-Up

A comprehensive learning repository covering React.js fundamentals, system design patterns, performance optimization, and frontend best practices. This project serves as a structured reference for interview preparation and advanced frontend development.

## Table of Contents

- [Project Structure](#project-structure)
- [1. React Brush-Up (Core Concepts)](#1-react-brush-up-core-concepts)
- [2. Frontend System Design Practice](#2-frontend-system-design-practice)
- [3. JSON Server (Mock API)](#3-json-server-mock-api)
- [Getting Started](#getting-started)
- [Technologies Used](#technologies-used)
- [Contributing](#contributing)

## Project Structure

```text
react-ocean/
├── brush-up-react/                    # React core concepts, hooks, optimization practice
├── frontend-system-design-practice/    # System design interview questions with implementations
├── json-server/                       # Mock REST API for practice
└── readme.md                          # This file
```

## 1. React Brush-Up (Core Concepts)

The [brush-up-react](./brush-up-react/) directory contains hands-on implementations of React fundamentals, hooks, and advanced concepts. It serves as a practical reference for understanding React's core features.

### Key Topics Covered:

- **React Fundamentals**: Components, JSX, Props, State, Events
- **React Hooks**: `useState`, `useEffect`, `useRef`, `useContext`, `useReducer`, `useMemo`, `useCallback`
- **Routing**: React Router DOM configuration and navigation
- **State Management**: Context API, Redux Toolkit (RTK)
- **Styling**: Tailwind CSS integration
- **TypeScript**: Type-safe React development
- **Performance**: React optimization techniques and patterns
- **Component Composition**: Parent-child patterns, lifting state up

[View Brush-Up React Documentation](./brush-up-react/README.md)

## 2. Frontend System Design Practice

The [frontend-system-design-practice](./frontend-system-design-practice/) directory contains real-world frontend system design interview questions with detailed explanations and working implementations. This is built with Next.js and focuses on scalable frontend architecture.

### System Design Topics Covered:

| # | Topic | Status | Documentation |
|---|------|--------|---------------|
| 1 | **Autocomplete Search with Debouncing & Caching** | Done | [View](./frontend-system-design-practice/src/components/search/) |
| 2 | **Infinite Scrolling Feed** | Done | [View](./frontend-system-design-practice/src/components/infiniteScroll/readme.md) |
| 3 | **React Application Optimization** | Done | [View](./frontend-system-design-practice/src/components/optimization/readme.md) |
| 4 | **Web Vitals (LCP, CLS, INP) Optimization** | Done | [View](./frontend-system-design-practice/src/components/improveWebVital/) |
| 5 | **Reusable Component Library for Large Teams** | Done | [View](./frontend-system-design-practice/src/components/componentLibrary/readme.md) |
| 6 | **API Failures, Retries & Graceful Handling** | Done | [View](./frontend-system-design-practice/src/components/apiFailureRetries/readme.md) |
| 7 | **Dark Mode Implementation** | In Progress | [View](./frontend-system-design-practice/src/components/day-07/) |
| 8 | **State Management: Redux vs Context vs Zustand** | Done | [View](./frontend-system-design-practice/src/components/day-07/readme.md) |
| 9 | **Frontend Security (XSS, CSRF, Token Leakage)** | Done | [View](./frontend-system-design-practice/src/components/day-09/frontend_security_architecture_readme.md) |
| 10 | **SSR/SSG & Hydration Strategy** | In Progress | [View](./frontend-system-design-practice/) |
| 11 | **Scalable Dashboard Design** | Planned | - |
| 12 | **Real-time Notification System** | Planned | - |
| 13 | **Frontend Architecture for Chat Apps** | Planned | - |
| 14 | **Feature Flag System** | Planned | - |
| 15 | **Client-side Caching Strategies** | Planned | - |
| 16 | **Analytics Event Tracking Pipeline** | Planned | - |
| 17 | **Error Logging & Monitoring** | Planned | - |

### Additional Topics:

- **Browser Security: Secure Cookies** - Done - [View Security Docs](./frontend-system-design-practice/src/components/day-09/)

[View System Design Practice Documentation](./frontend-system-design-practice/README.md)

## 3. JSON Server (Mock API)

The [json-server](./json-server/) directory provides a mock REST API for testing frontend applications without needing a real backend. It's used extensively with the system design practice examples to simulate real-world API interactions.

- **Purpose**: Mock API for autocomplete search, infinite scrolling, and other API-dependent features
- **Features**: RESTful endpoints, dynamic data filtering, pagination support
- **Integration**: Used with components to demonstrate API handling patterns

## Getting Started

### Prerequisites

- Node.js (v16 or higher)
- npm or yarn

### Installation

1. **Clone the repository**

   ```bash
   git clone https://github.com/mamun060/frontend-reactjs-nextjs-ocean.git
   cd frontend-reactjs-nextjs-ocean
   ```

2. **Set up Brush-Up React**

   ```bash
   cd brush-up-react
   npm install
   npm run dev
   ```

3. **Set up Frontend System Design Practice**

   ```bash
   cd frontend-system-design-practice
   npm install
   npm run dev
   ```

4. **Start Mock API (JSON Server)**

   ```bash
   cd json-server
   npm install
   npm start
   # or npx json-server --watch db.json --port 3001
   ```

## Technologies Used

### Core Technologies
- **React.js** - UI library
- **Next.js** - React framework for production
- **TypeScript** - Type safety
- **Tailwind CSS** - Utility-first CSS framework

### Libraries & Tools
- **React Router DOM** - Client-side routing
- **Redux Toolkit (RTK) & RTK Query** - State management & data fetching
- **Zustand** - Lightweight state management
- **JSON Server** - Mock REST API
- **DOMPurify** - XSS sanitization
- **Storybook** - Component library documentation
- **Turborepo/Nx** - Monorepo tooling (for component libraries)

## Contributing

This repository is for learning and interview preparation. If you find any issues or want to add new topics, feel free to open an issue or submit a pull request.

## Author

**Md Al Mamun** - [GitHub](https://github.com/mamun060)

---

*This repository is continuously updated with new system design patterns and React best practices.*