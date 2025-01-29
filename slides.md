---
# You can also start simply with 'default'
theme: dracula
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# some information about your slides (markdown enabled)
title: Full Stack Development Best Practices
info: |
  ## Full Stack Development Best Practices
  A comprehensive guide for junior developers.
# apply unocss classes to the current slide
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
highlighter: shiki
---

# Web Development Best Practices
React, Node.js, Express, and Friends


---
layout: default
---

# About Me

<div class="flex items-center gap-4">
<div>
<img src="https://github.com/talhabalaj.png" class="w-36 h-36" />
</div>

<div class="pt-1 pb-1">
<p class="text-2xl m-0 font-bold text-white">Talha Balaj</p>

- 🏢 Senior Software Engineer at mobileLive
- ⚛️ Full Stack Developer with expertise in React & Node.js  
- 🚀 Passionate about building scalable web applications
</div>
</div>

---
layout: default
---

# Agenda

<v-click>

- Improve app performance, security, and code quality.

</v-click>

<v-click>

- Identify common pitfalls ("footguns") in React & Node.js.

</v-click>

<v-click>

- Hands-on examples and better practices.

</v-click>

---
layout: section
---

# Quick Recap
React, Node.js, and Express Fundamentals

---
layout: two-cols-header
---

# React Fundamentals

A quick refresher on core React concepts before diving into best practices.


::left::
<div>
Components & Props

```jsx {all|none|none}
const Greeting = ({ name }) => {
  return <h1>Hello, {name}!</h1>;
};
```

useState Hook

```jsx {none|all|none}{at:1}
// useState Hook
const Counter = () => {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
};
```

</div>

::right::

<div class="ml-4">
useEffect Hook

```jsx {none|none|all}{at:1}
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);
  
  return user ? <div>{user.name}</div> : <div>Loading...</div>;
};
```
</div>
---
layout: default
---

# Node.js/Express Basics

<v-switch>

<template #0>
Express is a minimal and flexible Node.js web application framework that is widely used for building web applications and APIs.
</template>

<template #1-7>

> **Note**: Express is now in a **maintenance mode** and is **not actively developed**. Use Fastify, Hono, or Nest.js instead.
</template>

</v-switch>


```javascript {all|5|7-10|12-16|17-20}{at: 3}
const express = require('express');
const app = express();

// Middleware
app.use(express.json());

app.get('/api/users', async (req, res) => {
  const users = await db.users.findAll();
  res.json(users);
});

app.post('/api/users', async (req, res) => {
  const user = await db.users.create(req.body);
  res.status(201).json(user);
});

// Error Middleware
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: 'Server error' });
});
```

---
layout: section
---

# Performance
Common Pitfalls and Solutions

---
layout: default
---

# React Performance Footguns

1.  Inline Object Prop Causing Re-renders


````md magic-move
```jsx {all|8|3,7|8|14}{at: 1}
// ❌ Bad: New object created every render
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <ChildComponent style={{ color: "red" }} /> {/* New object every time! */}
    </div>
  );
}

const ChildComponent = ({ style }) => {
  console.log("Child re-rendered!"); // Logs on every Parent re-render
  return <div style={style}>Child</div>;
};
```
```jsx {all|4}
// ✅ Good: Memoized the inline object
function Parent() {
  const [count, setCount] = useState(0);
  const memoizedStyle = useMemo(() => ({ color: "red" }), []); // Cache the object

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <ChildComponent style={memoizedStyle} />
    </div>
  );
}

const ChildComponent = React.memo(({ style }) => { // Memoize the component
  console.log("Child re-rendered!"); // Only logs once
  return <div style={style}>Child</div>;
});
```

````
<v-switch at="-2">
<template #1-2>

**Why it re-renders:** 
The style prop is a new object on every Parent render, so ChildComponent re-renders even if nothing changed.
</template>

<template #3>

**Fix:** Use `useMemo` to memoize the object
</template>

</v-switch>


---

# Understanding useMemo

#### What is `useMemo`?
- **Purpose**: Memoizes expensive calculations to avoid re-computing on every render.  
- **Syntax**:  
  ```javascript
  const memoizedValue = useMemo(() => computeValue(), [dependencies]);
  ```    

<div class="mt-2" />

<v-click>

#### Dependency Array Rules
</v-click>

<v-click>

1. **No Array**: Re-computes **every render** (avoid this!).  
   ```javascript
   const value = useMemo(() => compute()); // ❌ Re-runs every time  
   ```  
</v-click>

<v-click>

2. **Empty Array**: Computes **once** (mounting).  
   ```javascript
   const value = useMemo(() => compute(), []); // ✅ Runs once  
   ```  
</v-click>

<v-click>

3. **With Dependencies**: Re-computes **only when dependencies change**.  
   ```javascript
   const value = useMemo(() => compute(a, b), [a, b]); // ✅ Re-runs if `a`/`b` change  
   ```  
</v-click>
---

#### Example: Filtering a List 
**Problem Without `useMemo`**:  
```javascript
const filteredList = list.filter(item => item.includes(searchQuery)); // Recomputes on every render  
```  
**Fix With `useMemo`**:  
```javascript
const filteredList = useMemo(
  () => list.filter(item => item.includes(searchQuery)),
  [list, searchQuery] // ✅ Only recompute when `list` or `searchQuery` changes  
);
```  

---

#### **Common Pitfalls**  
1. **Missing Dependencies**:  
   ```javascript
   const total = useMemo(() => a + b, [a]); // ❌ Missing `b` → stale value!  
   ```  
2. **Unnecessary Use**:  
   ```javascript
   const num = useMemo(() => 5, []); // ❌ Overkill for primitive values.  
   ```  
3. **Mutable Dependencies**:  
   ```javascript
   const config = { threshold: 10 };
   const result = useMemo(() => compute(config), [config]); // ❌ New object every render → recomputes!  
   ```  

---

#### **Key Takeaways**  
- **Use When**:  
  - Heavy computations (e.g., sorting/filtering large arrays).  
  - Preventing child re-renders due to unchanged props.  
- **Avoid When**:  
  - Calculations are trivial.  
  - Dependencies change too frequently.  
- **Always**:  
  - Include **all dependencies** used inside `useMemo`.  
  - Use the **React ESLint plugin** to catch missing dependencies.  

---

**Visual Tip**: Highlight dependencies in green (correct) and red (incorrect) in code examples!



---

# Better Reac
```
// ❌ Bad: Unnecessary effect for derived state
const BadCounter = () => {
  const [count, setCount] = useState(0);
  const [doubled, setDoubled] = useState(0);
  
  useEffect(() => {
    setDoubled(count * 2); // Unnecessary effect!
  }, [count]);
  
  return <div>{doubled}</div>;
};
```
```jsx


// ✅ Good: Direct calculation for derived state
const GoodCounter = () => {
  const [count, setCount] = useState(0);
  const doubled = count * 2; // Direct calculation
  
  return <div>{doubled}</div>;
};

// Code Splitting Example
const HeavyComponent = React.lazy(() => 
  import('./HeavyComponent')
);

// List Virtualization
import { FixedSizeList } from 'react-window';

const VirtualizedList = ({ items }) => (
  <FixedSizeList
    height={400}
    width={600}
    itemCount={items.length}
    itemSize={50}
  >
    {({ index, style }) => (
      <div style={style}>{items[index]}</div>
    )}
  </FixedSizeList>
);
```

---

# Node.js/API Performance Footguns

```javascript
// ❌ Bad: Blocking the event loop
app.get('/api/data', (req, res) => {
  const data = fs.readFileSync('large-file.json');
  const parsed = JSON.parse(data);
  res.json(parsed);
});

// ❌ Bad: Memory leak
let cache = {};
app.get('/api/cache', (req, res) => {
  cache[req.query.key] = req.query.value; // Never cleaned up!
  res.json({ success: true });
});

// ❌ Bad: N+1 Query Problem
app.get('/api/posts', async (req, res) => {
  const posts = await Post.findAll();
  // N+1 Problem: One query for posts + N queries for authors
  for (let post of posts) {
    post.author = await User.findByPk(post.authorId);
  }
  res.json(posts);
});
```

---

# Better Node.js/API Performance

```javascript
// ✅ Good: Streaming large files
app.get('/api/data', (req, res) => {
  const stream = fs.createReadStream('large-file.json');
  stream.pipe(res);
});

// ✅ Good: Proper caching with Redis
const Redis = require('ioredis');
const redis = new Redis();

app.get('/api/data', async (req, res) => {
  const cached = await redis.get('data-key');
  if (cached) return res.json(JSON.parse(cached));
  
  const data = await fetchData();
  await redis.setex('data-key', 3600, JSON.stringify(data));
  res.json(data);
});

// ✅ Good: Eager Loading to solve N+1
app.get('/api/posts', async (req, res) => {
  const posts = await Post.findAll({
    include: [{ model: User, as: 'author' }]
  });
  res.json(posts);
});

// ✅ Good: Pagination
app.get('/api/users', async (req, res) => {
  const { page = 1, limit = 10 } = req.query;
  const users = await User.findAndCountAll({
    limit: +limit,
    offset: (page - 1) * limit
  });
  res.json({
    users: users.rows,
    total: users.count,
    pages: Math.ceil(users.count / limit)
  });
});
```

---
layout: section
---

# Security
Common Vulnerabilities and Protections

---

# React Security Footguns

```jsx
// ❌ Bad: XSS Vulnerability
const DangerousComponent = ({ userContent }) => {
  return <div dangerouslySetInnerHTML={{ 
    __html: userContent 
  }} />;
};

// ❌ Bad: Exposed API Keys
const ApiComponent = () => {
  const API_KEY = "sk_live_123456"; // Exposed in client bundle!
  
  return <div>Protected Component</div>;
};

// ❌ Bad: Unescaped URL Parameters
const SearchComponent = () => {
  const [search, setSearch] = useState('');
  return (
    <a href={`/search?q=${search}`}>
      Search
    </a>
  );
};
```

---

# Better React Security

```jsx
// ✅ Good: Sanitized HTML
import DOMPurify from 'dompurify';

const SafeComponent = ({ userContent }) => {
  const sanitized = DOMPurify.sanitize(userContent);
  return <div dangerouslySetInnerHTML={{ 
    __html: sanitized 
  }} />;
};

// ✅ Good: Environment Variables
const ApiComponent = () => {
  const apiKey = process.env.REACT_APP_API_KEY;
  return <div>Protected Component</div>;
};

// ✅ Good: Encoded URL Parameters
const SafeSearchComponent = () => {
  const [search, setSearch] = useState('');
  return (
    <a href={`/search?q=${encodeURIComponent(search)}`}>
      Search
    </a>
  );
};
```

---

# Node.js/API Security Footguns

```javascript
// ❌ Bad: SQL Injection
app.get('/api/users', async (req, res) => {
  const { name } = req.query;
  const query = `SELECT * FROM users WHERE name = '${name}'`;
  const users = await db.raw(query);
  res.json(users);
});

// ❌ Bad: Hardcoded Secrets
const stripe = require('stripe')('sk_live_123456');

// ❌ Bad: Missing Security Headers
app.get('/api/data', (req, res) => {
  res.json({ sensitive: 'data' });
});
```

---

# Better Node.js/API Security

```javascript
// ✅ Good: Parameterized Queries with Sequelize
app.get('/api/users', async (req, res) => {
  const { name } = req.query;
  const users = await User.findAll({
    where: { name }
  });
  res.json(users);
});

// ✅ Good: Environment Variables
require('dotenv').config();
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

// ✅ Good: Security Middleware
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

app.use(helmet());
app.use(rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
}));

// CORS Configuration
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS.split(','),
  methods: ['GET', 'POST'],
  credentials: true
}));
```

---
layout: section
---

# Coding Practices
Writing Better Code

---

# React Code Smells

```jsx
// ❌ Bad: Prop Drilling
const App = () => {
  const [user, setUser] = useState(null);
  return (
    <Header user={user}>
      <Navigation user={user}>
        <Profile user={user} setUser={setUser}>
          <Settings user={user} setUser={setUser} />
        </Profile>
      </Navigation>
    </Header>
  );
};

// ❌ Bad: Giant Component
const MonolithComponent = () => {
  const [users, setUsers] = useState([]);
  const [posts, setPosts] = useState([]);
  const [comments, setComments] = useState([]);
  
  // Lots of useEffects
  useEffect(() => { /* fetch users */ }, []);
  useEffect(() => { /* fetch posts */ }, []);
  useEffect(() => { /* fetch comments */ }, []);
  
  // Lots of handlers
  const handleUserUpdate = () => { /* ... */ };
  const handlePostUpdate = () => { /* ... */ };
  const handleCommentUpdate = () => { /* ... */ };
  
  return (
    <div>
      {/* Hundreds of lines of JSX */}
    </div>
  );
};
```

---

# Better React Practices

```jsx
// ✅ Good: Context API
const UserContext = createContext();

const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
};

// ✅ Good: Custom Hooks
const useUsers = () => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchUsers()
      .then(data => {
        setUsers(data);
        setLoading(false);
      });
  }, []);
  
  return { users, loading };
};

// ✅ Good: Component Composition
const UserList = () => {
  const { users, loading } = useUsers();
  if (loading) return <Spinner />;
  return users.map(user => <UserCard key={user.id} user={user} />);
};

// ✅ Good: Testing Example
import { render, screen } from '@testing-library/react';

test('renders user list', async () => {
  render(<UserList />);
  expect(screen.getByText('Loading...')).toBeInTheDocument();
  const users = await screen.findAllByRole('listitem');
  expect(users).toHaveLength(3);
});
```

---

# Node.js Code Smells

```javascript
// ❌ Bad: Callback Hell
app.get('/api/data', (req, res) => {
  db.users.findOne({ id: req.query.id }, (err, user) => {
    if (err) return res.status(500).json({ error: err });
    db.posts.find({ userId: user.id }, (err, posts) => {
      if (err) return res.status(500).json({ error: err });
      db.comments.find({ postIds: posts.map(p => p.id) }, (err, comments) => {
        if (err) return res.status(500).json({ error: err });
        res.json({ user, posts, comments });
      });
    });
  });
});

// ❌ Bad: No Error Handling
app.post('/api/users', (req, res) => {
  User.create(req.body)
    .then(user => res.json(user));
  // No error handling!
});
```

---

# Better Node.js Practices

```javascript
// ✅ Good: Async/Await with Try/Catch
app.get('/api/data', async (req, res, next) => {
  try {
    const user = await db.users.findOne({ id: req.query.id });
    const posts = await db.posts.find({ userId: user.id });
    const comments = await db.comments.find({
      postIds: posts.map(p => p.id)
    });
    res.json({ user, posts, comments });
  } catch (error) {
    next(error);
  }
});

// ✅ Good: Error Handling Middleware
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith('4') ? 'fail' : 'error';
  }
}

// Global Error Handler
app.use((err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';

  if (process.env.NODE_ENV === 'development') {
    res.status(err.statusCode).json({
      status: err.status,
      error: err,
      message: err.message,
      stack: err.stack
    });
  } else {
    res.status(err.statusCode).json({
      status: err.status,
      message: err.message
    });
  }
});

// ✅ Good: Configuration Management
const config = require('config');

app.get('/api/config', (req, res) => {
  res.json({
    database: config.get('database.url'),
    apiVersion: config.get('api.version'),
    features: config.get('features')
  });
});
```

---
layout: center
class: "text-center"
---

# Questions?

Let's discuss your specific challenges!

Resources:
- 📚 [React Docs](https://react.dev)
- 📘 [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- 🛡️ [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- 🔧 [Performance Tools](https://web.dev/measure)

---
layout: section
---

# State Management
Best Practices & Common Patterns

---
layout: two-cols
---

# Local vs Global State

```jsx
// ✅ Local State: Use for component-specific data
const TodoItem = () => {
  const [isEditing, setIsEditing] = useState(false);
  const [draft, setDraft] = useState('');
  
  return isEditing ? (
    <input value={draft} onChange={e => setDraft(e.target.value)} />
  ) : (
    <div onClick={() => setIsEditing(true)}>...</div>
  );
};

// ✅ Global State: Use for shared data
import { create } from 'zustand';

const useStore = create((set) => ({
  todos: [],
  addTodo: (todo) => set((state) => ({ 
    todos: [...state.todos, todo] 
  })),
  removeTodo: (id) => set((state) => ({
    todos: state.todos.filter(todo => todo.id !== id)
  }))
}));
```

::right::

# When to Use What

- **Local State (useState)**
  - Form inputs
  - UI state (modals, toggles)
  - Component-specific data
  
- **Context API**
  - Theme data
  - User preferences
  - Authentication state
  - Localization data
  
- **Global State (Redux/Zustand)**
  - Complex app state
  - Shared data between routes
  - Server cache state
  - Real-time data

---

# Advanced React Patterns

```jsx
// ✅ Compound Components Pattern
const Select = ({ children, onValueChange }) => {
  const [isOpen, setIsOpen] = useState(false);
  const [value, setValue] = useState(null);
  
  return (
    <SelectContext.Provider value={{ isOpen, setIsOpen, value, setValue }}>
      {children}
    </SelectContext.Provider>
  );
};

Select.Trigger = () => {
  const { isOpen, setIsOpen, value } = useSelectContext();
  return (
    <button onClick={() => setIsOpen(!isOpen)}>
      {value || 'Select...'}
    </button>
  );
};

Select.Options = ({ children }) => {
  const { isOpen } = useSelectContext();
  if (!isOpen) return null;
  return <div className="options">{children}</div>;
};

Select.Option = ({ value, children }) => {
  const { setValue, setIsOpen } = useSelectContext();
  return (
    <div onClick={() => {
      setValue(value);
      setIsOpen(false);
    }}>
      {children}
    </div>
  );
};

// Usage
<Select onValueChange={console.log}>
  <Select.Trigger />
  <Select.Options>
    <Select.Option value="1">Option 1</Select.Option>
    <Select.Option value="2">Option 2</Select.Option>
  </Select.Options>
</Select>
```

---

# React Performance Deep Dive

```jsx
// ✅ Optimizing Re-renders
const ExpensiveList = React.memo(({ items }) => {
  return items.map(item => <Item key={item.id} {...item} />);
});

// ✅ Optimizing Callbacks
const TodoList = () => {
  const [todos, setTodos] = useState([]);
  
  const addTodo = useCallback((text) => {
    setTodos(prev => [...prev, { id: Date.now(), text }]);
  }, []);
  
  const removeTodo = useCallback((id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  }, []);
  
  return (
    <div>
      <AddTodoForm onAdd={addTodo} />
      <TodoItems items={todos} onRemove={removeTodo} />
    </div>
  );
};

// ✅ Optimizing Expensive Calculations
const DataGrid = ({ data, filters }) => {
  const filteredData = useMemo(() => {
    return data.filter(item => {
      return Object.entries(filters).every(([key, value]) => 
        item[key].includes(value)
      );
    });
  }, [data, filters]);
  
  return <Table data={filteredData} />;
};

// ✅ Intersection Observer for Lazy Loading
const LazyImage = ({ src, alt }) => {
  const [isVisible, setIsVisible] = useState(false);
  const imgRef = useRef();
  
  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => setIsVisible(entry.isIntersecting)
    );
    
    if (imgRef.current) {
      observer.observe(imgRef.current);
    }
    
    return () => observer.disconnect();
  }, []);
  
  return (
    <img
      ref={imgRef}
      src={isVisible ? src : 'placeholder.jpg'}
      alt={alt}
    />
  );
};
```

---

# Node.js Advanced Patterns

```javascript
// ✅ Repository Pattern
class UserRepository {
  async findById(id) {
    return await User.findByPk(id);
  }
  
  async create(data) {
    return await User.create(data);
  }
  
  async update(id, data) {
    const user = await this.findById(id);
    if (!user) throw new NotFoundError('User not found');
    return await user.update(data);
  }
}

// ✅ Service Layer Pattern
class AuthService {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }
  
  async register(userData) {
    const hashedPassword = await bcrypt.hash(userData.password, 10);
    const user = await this.userRepository.create({
      ...userData,
      password: hashedPassword
    });
    return this.generateToken(user);
  }
  
  async login(email, password) {
    const user = await this.userRepository.findByEmail(email);
    if (!user) throw new AuthError('Invalid credentials');
    
    const isValid = await bcrypt.compare(password, user.password);
    if (!isValid) throw new AuthError('Invalid credentials');
    
    return this.generateToken(user);
  }
  
  generateToken(user) {
    return jwt.sign(
      { id: user.id, email: user.email },
      process.env.JWT_SECRET,
      { expiresIn: '24h' }
    );
  }
}

// ✅ Middleware Factory Pattern
const validate = (schema) => {
  return (req, res, next) => {
    const { error } = schema.validate(req.body);
    if (error) {
      return res.status(400).json({
        error: error.details[0].message
      });
    }
    next();
  };
};

// Usage
app.post('/api/users', 
  validate(userSchema),
  async (req, res) => {
    // Handler code
  }
);
```

---

# Advanced Database Patterns

```javascript
// ✅ Transaction Management
async function transferMoney(fromId, toId, amount) {
  const t = await sequelize.transaction();
  
  try {
    const from = await Account.findByPk(fromId, { transaction: t });
    const to = await Account.findByPk(toId, { transaction: t });
    
    if (from.balance < amount) {
      throw new Error('Insufficient funds');
    }
    
    await from.decrement('balance', { by: amount, transaction: t });
    await to.increment('balance', { by: amount, transaction: t });
    
    await t.commit();
  } catch (error) {
    await t.rollback();
    throw error;
  }
}

// ✅ Optimistic Locking
const Comment = sequelize.define('Comment', {
  content: DataTypes.TEXT,
  version: {
    type: DataTypes.INTEGER,
    defaultValue: 0
  }
});

async function updateComment(id, content, version) {
  const [updated] = await Comment.update(
    { 
      content,
      version: version + 1
    },
    {
      where: {
        id,
        version
      }
    }
  );
  
  if (!updated) {
    throw new Error('Comment was modified by another user');
  }
}

// ✅ Efficient Batch Operations
async function batchCreateUsers(users) {
  const chunks = _.chunk(users, 1000);
  const results = [];
  
  for (const chunk of chunks) {
    const created = await User.bulkCreate(chunk, {
      returning: true,
      validate: true
    });
    results.push(...created);
  }
  
  return results;
}
```

---

# Testing Best Practices

```javascript
// ✅ React Component Testing
describe('UserProfile', () => {
  it('displays user data correctly', async () => {
    // Arrange
    const user = { id: 1, name: 'John' };
    const mockFetch = jest.fn().mockResolvedValue(user);
    global.fetch = mockFetch;
    
    // Act
    render(<UserProfile userId={1} />);
    
    // Assert
    expect(screen.getByText('Loading...')).toBeInTheDocument();
    expect(await screen.findByText('John')).toBeInTheDocument();
    expect(mockFetch).toHaveBeenCalledWith('/api/users/1');
  });
  
  it('handles error states', async () => {
    // Arrange
    const mockFetch = jest.fn().mockRejectedValue(new Error('Failed'));
    global.fetch = mockFetch;
    
    // Act
    render(<UserProfile userId={1} />);
    
    // Assert
    expect(await screen.findByText('Error loading user')).toBeInTheDocument();
  });
});

// ✅ API Integration Testing
describe('User API', () => {
  let app;
  
  beforeAll(async () => {
    await db.connect();
    app = createApp();
  });
  
  afterAll(async () => {
    await db.disconnect();
  });
  
  beforeEach(async () => {
    await db.clear();
  });
  
  it('creates a user successfully', async () => {
    const res = await request(app)
      .post('/api/users')
      .send({
        name: 'John',
        email: 'john@example.com'
      });
      
    expect(res.status).toBe(201);
    expect(res.body).toMatchObject({
      name: 'John',
      email: 'john@example.com'
    });
  });
  
  it('validates user input', async () => {
    const res = await request(app)
      .post('/api/users')
      .send({
        name: 'John'
        // Missing required email
      });
      
    expect(res.status).toBe(400);
    expect(res.body.error).toContain('email is required');
  });
});
```

---
layout: section
---

# API Design Patterns

---

# RESTful API Best Practices

```javascript
// ✅ Versioning
app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);

// ✅ Resource Naming
// Good: /api/users/{id}/posts
// Bad: /api/getUserPosts/{id}

// ✅ Query Parameters
app.get('/api/posts', (req, res) => {
  const {
    page = 1,
    limit = 10,
    sort = 'createdAt',
    order = 'desc',
    fields = '',
    search = ''
  } = req.query;
  
  // Build query
  const query = {
    offset: (page - 1) * limit,
    limit: +limit,
    order: [[sort, order]],
    where: search ? {
      [Op.or]: [
        { title: { [Op.iLike]: `%${search}%` } },
        { content: { [Op.iLike]: `%${search}%` } }
      ]
    } : undefined,
    attributes: fields ? fields.split(',') : undefined
  };
  
  // Execute query
  const posts = await Post.findAndCountAll(query);
  
  // Send response with metadata
  res.json({
    data: posts.rows,
    meta: {
      total: posts.count,
      pages: Math.ceil(posts.count / limit),
      current_page: +page,
      per_page: +limit
    }
  });
});

// ✅ Error Responses
class APIError extends Error {
  constructor(message, statusCode, errors = []) {
    super(message);
    this.statusCode = statusCode;
    this.errors = errors;
  }
  
  toJSON() {
    return {
      error: {
        message: this.message,
        code: this.statusCode,
        details: this.errors
      }
    };
  }
}

// Usage
app.post('/api/users', async (req, res, next) => {
  try {
    const validation = userSchema.validate(req.body);
    if (validation.error) {
      throw new APIError(
        'Validation failed',
        400,
        validation.error.details
      );
    }
    // ... rest of handler
  } catch (error) {
    next(error);
  }
});
```

---

# Advanced Security Practices

```javascript
// ✅ JWT with Refresh Tokens
class AuthController {
  async login(req, res) {
    const { email, password } = req.body;
    const user = await User.findOne({ email });
    
    if (!user || !await user.comparePassword(password)) {
      throw new AuthError('Invalid credentials');
    }
    
    const [accessToken, refreshToken] = await Promise.all([
      this.generateAccessToken(user),
      this.generateRefreshToken(user)
    ]);
    
    // Store refresh token hash
    await RefreshToken.create({
      userId: user.id,
      token: crypto.createHash('sha256').update(refreshToken).digest('hex'),
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) // 7 days
    });
    
    res.cookie('refreshToken', refreshToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
    });
    
    res.json({ accessToken });
  }
  
  async refresh(req, res) {
    const { refreshToken } = req.cookies;
    if (!refreshToken) {
      throw new AuthError('Refresh token required');
    }
    
    const tokenHash = crypto
      .createHash('sha256')
      .update(refreshToken)
      .digest('hex');
      
    const token = await RefreshToken.findOne({
      token: tokenHash,
      expiresAt: { [Op.gt]: new Date() }
    });
    
    if (!token) {
      throw new AuthError('Invalid refresh token');
    }
    
    const user = await User.findByPk(token.userId);
    const accessToken = await this.generateAccessToken(user);
    
    res.json({ accessToken });
  }
}

// ✅ Rate Limiting with Redis
const rateLimiter = new RateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rate-limit:'
  }),
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  handler: (req, res) => {
    throw new APIError('Too many requests', 429);
  }
});

// ✅ Security Headers
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'", process.env.API_URL],
      fontSrc: ["'self'", 'https://fonts.gstatic.com'],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"],
    },
  },
  crossOriginEmbedderPolicy: true,
  crossOriginOpenerPolicy: true,
  crossOriginResourcePolicy: { policy: "same-site" },
  dnsPrefetchControl: true,
  frameguard: { action: "deny" },
  hidePoweredBy: true,
  hsts: true,
  ieNoOpen: true,
  noSniff: true,
  referrerPolicy: { policy: "strict-origin-when-cross-origin" },
  xssFilter: true,
}));
```

---
layout: section
---

# Deployment & DevOps Best Practices

---

# CI/CD Pipeline Example

```yaml
# .github/workflows/main.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
          
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '16'
        cache: 'npm'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run linter
      run: npm run lint
      
    - name: Run tests
      run: npm test
      env:
        DATABASE_URL: postgresql://test:test@localhost:5432/test
        
    - name: Build
      run: npm run build
      
  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Deploy to production
      uses: digitalocean/action-doctl@v2
      with:
        token: ${{ secrets.DIGITALOCEAN_ACCESS_TOKEN }}
        
    - name: Update deployment
      run: doctl apps update ${{ secrets.APP_ID }} --spec .do/app.yaml
```

---

# Docker Best Practices

```dockerfile
# Multi-stage build for Node.js application
FROM node:16-alpine AS builder

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci

# Copy source and build
COPY . .
RUN npm run build

# Production image
FROM node:16-alpine

WORKDIR /app

# Copy only production dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy built assets from builder
COPY --from=builder /app/dist ./dist

# Set environment variables
ENV NODE_ENV=production
ENV PORT=3000

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --quiet --tries=1 --spider http://localhost:3000/health || exit 1

# Start application
CMD ["node", "dist/main.js"]
```

---

# Monitoring & Logging

```javascript
// ✅ Winston Logger Setup
const winston = require('winston');
require('winston-daily-rotate-file');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'api-service' },
  transports: [
    new winston.transports.DailyRotateFile({
      filename: 'logs/error-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      level: 'error',
      maxFiles: '14d'
    }),
    new winston.transports.DailyRotateFile({
      filename: 'logs/combined-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      maxFiles: '14d'
    })
  ]
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.combine(
      winston.format.colorize(),
      winston.format.simple()
    )
  }));
}

// ✅ Request Logging Middleware
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    logger.info('Request completed', {
      method: req.method,
      url: req.url,
      status: res.statusCode,
      duration,
      userAgent: req.get('user-agent'),
      ip: req.ip
    });
  });
  
  next();
});

// ✅ Performance Monitoring
const prometheus = require('prom-client');
const collectDefaultMetrics = prometheus.collectDefaultMetrics;

// Enable default metrics
collectDefaultMetrics({ prefix: 'api_' });

// Custom metrics
const httpRequestDurationMicroseconds = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', prometheus.register.contentType);
  res.send(await prometheus.register.metrics());
});
```

---
layout: center
class: "text-center"
---

# Thank You!

Let's build better applications together!

Connect with me:
- 🌐 [GitHub](https://github.com/talhabalaj)
- 💼 [LinkedIn](https://linkedin.com/in/talhabalaj)
- 🐦 [Twitter](https://twitter.com/talhabalaj)

