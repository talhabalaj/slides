### **2-Hour React & Node.js/Express Workshop: Performance, Security, Coding Practices**  
**Audience:** Junior Developers  

---

#### **Slide 1: Introduction (5 mins)**  
- Welcome & session goals:  
  - Improve app performance, security, and code quality.  
  - Identify common pitfalls ("footguns") in React & Node.js.  
  - Hands-on examples and better practices.  

---

#### **Slide 2: Quick Recap (5 mins)**  
- **React**: Components, hooks (useState, useEffect), state management.  
- **Node.js/Express**: Middleware, REST APIs, async operations.  

---

### **Section 1: Performance (35 mins)**  

#### **Slide 3: React Performance Footguns**  
- Unnecessary re-renders (e.g., new object props every render).  
- Large bundle sizes (no code splitting).  
- Overusing `useEffect` for derived state.  
- **Demo**: Slow component due to inline object props.  

#### **Slide 4: Better React Performance**  
- Memoization: `React.memo`, `useMemo`, `useCallback`.  
- Code splitting: `React.lazy`, dynamic imports.  
- Virtualize long lists (e.g., `react-window`).  
- **Fix**: Optimize the demo with `useMemo`.  

#### **Slide 5: Node.js/API Performance Footguns**  
- Blocking the event loop (sync operations in async routes).  
- Memory leaks (unclosed streams, timers).  
- N+1 database queries.  
- **Demo**: Blocking endpoint with `JSON.parse(fs.readFileSync)`.  

#### **Slide 6: Better Node.js/API Performance**  
- Async/await for non-blocking code.  
- Stream large datasets (e.g., `fs.createReadStream`).  
- Caching (Redis), pagination, compression (middleware like `compression`).  
- **Fix**: Replace `readFileSync` with async + streams.  

---

### **Section 2: Security (35 mins)**  

#### **Slide 7: React Security Footguns**  
- XSS via `dangerouslySetInnerHTML` or unsanitized user input.  
- Exposing sensitive data in client-side code (e.g., API keys).  
- **Demo**: XSS vulnerability via user-generated content.  

#### **Slide 8: Better React Security**  
- Sanitize inputs with libraries like `DOMPurify`.  
- Avoid inline scripts, use CSP headers.  
- Environment variables for client-side secrets (via build tools).  
- **Fix**: Sanitize the XSS demo input.  

#### **Slide 9: Node.js/API Security Footguns**  
- SQL/NoSQL injection (e.g., concatenating user input into queries).  
- Hardcoded secrets in code.  
- Missing CORS, rate limiting, or HTTPS.  
- **Demo**: SQL injection in a raw query.  

#### **Slide 10: Better Node.js/API Security**  
- Use ORMs (e.g., Sequelize) or parameterized queries.  
- Environment variables + `dotenv` for secrets.  
- Middleware: `helmet`, `cors`, `express-rate-limit`.  
- **Fix**: Replace raw query with parameterized Sequelize call.  

---

### **Section 3: Coding Practices (25 mins)**  

#### **Slide 11: React Code Smells**  
- Prop drilling vs. overusing Redux.  
- Giant components with mixed logic.  
- Ignoring ESLint/TypeScript errors.  
- **Demo**: Prop drilling hell.  

#### **Slide 12: Better React Practices**  
- State management: Context for small apps, Redux/Zustand for complex ones.  
- Custom hooks to encapsulate logic.  
- Testing: Jest + React Testing Library.  
- **Fix**: Refactor prop drilling with Context.  

#### **Slide 13: Node.js Code Smells**  
- Callback hell (nested `.then`/`.catch`).  
- No error handling (uncaught promise rejections).  
- Mixing environments (e.g., dev config in prod).  
- **Demo**: Unhandled promise rejection crashing the server.  

#### **Slide 14: Better Node.js Practices**  
- Async/await with `try/catch`.  
- Centralized error handling middleware.  
- Config management with `config` or `dotenv`.  
- **Fix**: Add error middleware to catch unhandled rejections.  

---

#### **Slide 15: General Best Practices**  
- Code reviews, pair programming.  
- Documentation (Swagger for APIs, Storybook for UI).  
- Monitoring (logging, APM tools).  

---

#### **Slide 16: Q&A + Next Steps (15 mins)**  
- Open floor for questions.  
- **Resources**:  
  - React: React Docs, Kent C. Dodds’ blog.  
  - Node.js: Express Best Practices, Node.js Security Checklist.  
  - Tools: Lighthouse, SonarQube, Postman.  

---

**Total Time:** 120 minutes.  
**Delivery Tips:** Use live coding demos for footguns/fixes, encourage interactive debugging, and share code snippets via GitHub/Gist.