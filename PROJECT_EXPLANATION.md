# 📚 Syncrify - Complete Project Explanation for Beginners

## 🎯 What is This Project?

**Syncrify** is a real-time collaborative document editing application (like Google Docs). Multiple users can:
- Create documents
- Edit documents together in real-time
- See who else is editing
- Add collaborators to documents
- All changes sync instantly across all users

---

## 🏗️ Project Architecture Overview

This project uses the **MERN Stack**:

1. **M**ongoDB - Database (stores users and documents)
2. **E**xpress.js - Backend framework (handles API requests)
3. **R**eact - Frontend library (user interface)
4. **N**ode.js - JavaScript runtime (runs the server)

**Plus:**
- **Socket.IO** - Real-time communication (enables live collaboration)
- **Quill** - Rich text editor (the document editor you see)

---

## 📁 Project Structure Explained

```
SYNCRIFY-master/
│
├── src/                          # Main source code folder
│   │
│   ├── server.js                 # 🚀 Entry point - starts the server
│   │
│   ├── package.json              # Backend dependencies
│   │
│   ├── .env                      # Environment variables (secrets)
│   │
│   ├── models/                   # 📊 Database schemas (data structure)
│   │   ├── user.model.js         # User data structure
│   │   └── documents.model.js    # Document data structure
│   │
│   ├── controllers/              # 🎮 Business logic (what happens when API is called)
│   │   ├── auth.controller.js   # Login, register, verify email
│   │   ├── doc.controller.js    # Create, read, update, delete documents
│   │   └── socket.controller.js  # Real-time collaboration logic
│   │
│   ├── routes/                   # 🛣️ API endpoints (URLs)
│   │   ├── user.routes.js        # /api/v1/users/* routes
│   │   └── document.routes.js   # /api/v1/documents/* routes
│   │
│   ├── middlewares/              # 🔒 Security checks
│   │   └── auth.middleware.js    # Verifies user is logged in
│   │
│   ├── utils/                    # 🛠️ Helper functions
│   │   ├── dbConnect.js          # Connects to MongoDB
│   │   ├── email.js              # Sends verification emails
│   │   └── emailTemplate.js      # Email HTML templates
│   │
│   └── client/                   # 🎨 Frontend (React app)
│       │
│       ├── package.json           # Frontend dependencies
│       │
│       ├── src/
│       │   ├── main.jsx          # React entry point
│       │   ├── App.jsx           # Main app component (routing)
│       │   │
│       │   ├── pages/            # 📄 Different pages/screens
│       │   │   ├── auth/
│       │   │   │   ├── Login.jsx
│       │   │   │   └── Register.jsx
│       │   │   └── main/
│       │   │       ├── DocumentHome.jsx  # List of all documents
│       │   │       ├── EditDocument.jsx  # Document editor page
│       │   │       └── Editor.jsx        # Quill editor component
│       │   │
│       │   ├── components/       # 🧩 Reusable UI components
│       │   │   ├── Navbar.jsx
│       │   │   ├── Card.jsx
│       │   │   ├── Modal.jsx
│       │   │   └── Loader.jsx
│       │   │
│       │   ├── context/          # 🌐 Global state management
│       │   │   ├── authContext.jsx      # User login state
│       │   │   └── supplierContext.jsx  # Document & socket state
│       │   │
│       │   └── helpers/          # 🛠️ Helper functions
│       │       ├── auth/
│       │       │   └── auth.helper.js   # Login/register API calls
│       │       ├── docs/
│       │       │   └── doc.helper.js    # Document API calls
│       │       ├── routes/
│       │       │   └── PrivateRoutes.jsx  # Protected routes
│       │       └── config.js             # API URL configuration
```

---

## 🔄 How Data Flows Through the Application

### 1. **User Registration Flow**

```
User fills form → Register.jsx
    ↓
auth.helper.js → POST /api/v1/users/register
    ↓
auth.controller.js → register() function
    ↓
- Hash password with bcrypt
- Create user in MongoDB
- Generate verification token
- Send email via nodemailer
    ↓
User receives email → Clicks link
    ↓
GET /api/v1/users/verifyemail/:tokenId
    ↓
User is verified → Can now login
```

### 2. **User Login Flow**

```
User enters credentials → Login.jsx
    ↓
auth.helper.js → POST /api/v1/users/login
    ↓
auth.controller.js → login() function
    ↓
- Check if user exists
- Check if email verified
- Compare password with bcrypt
- Generate JWT token
    ↓
Token saved in localStorage
    ↓
authContext.jsx → Updates global auth state
    ↓
User redirected to /home
```

### 3. **Document Creation Flow**

```
User clicks "Create New" → DocumentHome.jsx
    ↓
User enters title → Modal form
    ↓
doc.helper.js → POST /api/v1/documents
    ↓
auth.middleware.js → Validates JWT token
    ↓
doc.controller.js → createDocument()
    ↓
- Check if title exists
- Check document limit (max 3)
- Create document in MongoDB
    ↓
Document appears in list
```

### 4. **Real-Time Editing Flow**

```
User opens document → EditDocument.jsx
    ↓
Editor.jsx → Initializes Quill editor
    ↓
Socket.IO connects → socket.controller.js
    ↓
User joins room → socket.emit('joinRoom')
    ↓
User types → Quill detects change
    ↓
socket.emit('send-changes') → Sends delta (change)
    ↓
Server receives → socket.on('send-changes')
    ↓
Server broadcasts → io.to(roomId).emit('receive-changes')
    ↓
Other users receive → socket.on('receive-changes')
    ↓
Quill updates → quill.updateContents(delta)
    ↓
All users see the change instantly!
```

---

## 🗄️ Database Models Explained

### User Model (`user.model.js`)

```javascript
{
  username: "john_doe",        // Unique username
  email: "john@example.com",   // Unique email
  password: "hashed_password", // Encrypted with bcrypt
  verificationToken: "abc123", // For email verification
  isVerified: true,            // Email verified?
  personalInfo: "",            // Optional info
  timestamps: true             // Auto: createdAt, updatedAt
}
```

### Document Model (`documents.model.js`)

```javascript
{
  title: "My Document",        // Document name
  content: { ... },            // Quill editor content (Delta format)
  owner: ObjectId("..."),     // Reference to User who created it
  collaborators: [            // Array of User IDs
    ObjectId("..."),
    ObjectId("...")
  ],
  isPublic: false,            // Public or private?
  timestamps: true            // Auto: createdAt, updatedAt
}
```

---

## 🔌 API Endpoints Explained

### User Routes (`/api/v1/users`)

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/register` | Create new account | ❌ No |
| GET | `/verifyemail/:tokenId` | Verify email address | ❌ No |
| POST | `/login` | Login user | ❌ No |
| GET | `/me` | Get current user info | ✅ Yes |

### Document Routes (`/api/v1/documents`)

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/` | Get all user's documents | ✅ Yes |
| POST | `/` | Create new document | ✅ Yes |
| GET | `/:documentId` | Get single document | ✅ Yes |
| PATCH | `/:documentId` | Update document | ✅ Yes |
| DELETE | `/:documentId` | Delete document | ✅ Yes |
| PATCH | `/add-collaborator/:documentId` | Add collaborator | ✅ Yes |
| GET | `/get-all-collaborators/:documentId` | Get collaborators | ✅ Yes |

---

## 🎨 Frontend Components Explained

### 1. **App.jsx** - Main Router
- Sets up React Router
- Defines all routes:
  - `/` → Login page
  - `/register` → Register page
  - `/home` → Document list (protected)
  - `/edit/:id` → Document editor (protected)

### 2. **AuthContext** - Global Auth State
- Stores: `{ user, token }`
- Used throughout app to check if user is logged in
- Persists in localStorage with 60-minute expiry

### 3. **SupplierContext** - Global App State
- Stores:
  - `currentDoc` - Currently open document
  - `quill` - Quill editor instance
  - `socket` - Socket.IO connection
  - `darkMode` - Theme preference
  - `loading` - Loading state

### 4. **Login.jsx** - Login Page
- Form with email & password
- Calls `login()` helper
- On success: saves token, updates auth context, redirects to `/home`

### 5. **Register.jsx** - Registration Page
- Form with username, email, password
- Validates: username 3-10 chars, password min 6 chars
- Calls `register()` helper
- Sends verification email

### 6. **DocumentHome.jsx** - Document List
- Shows greeting with username
- Displays all documents user owns or collaborates on
- "Create New" button opens modal
- Each document shown as Card component
- Can delete documents

### 7. **EditDocument.jsx** - Document Editor Page
- Left sidebar: Shows online collaborators & all collaborators
- Main area: Quill editor
- Real-time features:
  - Joins Socket.IO room when document opens
  - Sends changes when user types
  - Receives changes from other users
  - Auto-saves every 30 seconds
- Can add new collaborators

### 8. **Editor.jsx** - Quill Editor Component
- Initializes Quill editor
- Configures toolbar (headers, bold, italic, etc.)
- Enables cursor tracking (quill-cursors)
- Applies dark/light theme
- Disabled until document loads

---

## 🔐 Authentication & Security

### JWT (JSON Web Token)
- When user logs in, server creates a JWT token
- Token contains: `{ email, id }`
- Token expires in 1 hour
- Token sent in header: `Authorization: Bearer <token>`

### Auth Middleware (`auth.middleware.js`)
- Runs before protected routes
- Extracts token from header
- Verifies token with `JWT_SECRET`
- Adds `req.user = { email, id }` to request
- If invalid → Returns 401 Unauthorized

### Private Routes (`PrivateRoutes.jsx`)
- Wraps protected pages (`/home`, `/edit/:id`)
- Checks if token exists in localStorage
- If no token → Redirects to `/login`
- If token exists → Shows the page

---

## ⚡ Socket.IO Real-Time Communication

### Socket Events Explained

#### Client → Server Events:

1. **`joinRoom`** - User joins a document room
   ```javascript
   socket.emit('joinRoom', { roomId: docId, username: 'john' })
   ```
   - Server adds user to room
   - Notifies other users: `someoneJoined`

2. **`leaveRoom`** - User leaves a document room
   ```javascript
   socket.emit('leaveRoom', { roomId: docId, username: 'john' })
   ```
   - Server removes user from room
   - Notifies other users: `someoneLeft`

3. **`get-doc`** - Request document content
   ```javascript
   socket.emit('get-doc', { docId: docId })
   ```
   - Server fetches document from database
   - Sends back: `load-document` event

4. **`send-changes`** - Send text changes
   ```javascript
   socket.emit('send-changes', { delta: change, roomId: docId, username: 'john' })
   ```
   - Server broadcasts to all users in room
   - Others receive: `receive-changes` event

5. **`save-doc`** - Save document to database
   ```javascript
   socket.emit('save-doc', { docId: docId, data: quill.getContents() })
   ```
   - Server saves to MongoDB
   - Auto-saves every 30 seconds

#### Server → Client Events:

1. **`someoneJoined`** - Someone joined the room
   - Updates online collaborators list

2. **`someoneLeft`** - Someone left the room
   - Updates online collaborators list

3. **`load-document`** - Document content loaded
   - Quill editor displays the content

4. **`receive-changes`** - Received changes from another user
   - Quill applies the changes

---

## 🎯 Key Technologies Deep Dive

### 1. **React**
- Component-based UI library
- Uses hooks: `useState`, `useEffect`, `useContext`
- React Router for navigation
- Context API for global state

### 2. **Express.js**
- Web framework for Node.js
- Handles HTTP requests (GET, POST, PATCH, DELETE)
- Middleware for authentication
- RESTful API design

### 3. **MongoDB + Mongoose**
- NoSQL database (stores JSON-like documents)
- Mongoose = ODM (Object Document Mapper)
- Schemas define data structure
- Queries: `find()`, `create()`, `findByIdAndUpdate()`

### 4. **Socket.IO**
- WebSocket library for real-time communication
- Bidirectional: client ↔ server
- Rooms: Group users by document
- Events: Custom messages between client/server

### 5. **Quill**
- Rich text editor (WYSIWYG)
- Delta format: Represents changes as operations
- Example delta: `{ ops: [{ insert: 'Hello' }] }`
- Modules: toolbar, cursors, etc.

### 6. **bcrypt**
- Password hashing library
- One-way encryption (can't decrypt)
- Compare: `bcrypt.compare(password, hashedPassword)`

### 7. **JWT (jsonwebtoken)**
- Token-based authentication
- Signed with secret key
- Contains user info (email, id)
- Stateless (no server session needed)

### 8. **Nodemailer**
- Sends emails from Node.js
- Uses Gmail SMTP
- Sends verification links

---

## 🔄 Complete User Journey Example

### Scenario: Two users collaborate on a document

1. **User A creates document:**
   - Clicks "Create New" → Enters title "Project Plan"
   - Document saved to MongoDB
   - User A sees it in their list

2. **User A adds User B as collaborator:**
   - Opens document
   - Clicks "Add Collaborator"
   - Enters User B's email
   - User B added to `collaborators` array

3. **User A opens document:**
   - Clicks on document card
   - Goes to `/edit/:id`
   - Socket.IO connects
   - Emits `joinRoom` → Joins room
   - Emits `get-doc` → Loads content
   - Quill editor displays document

4. **User B opens same document:**
   - Opens document
   - Socket.IO connects
   - Emits `joinRoom` → Joins same room
   - User A sees "User B joined" notification
   - Both see each other in "Online Collaborators"

5. **User A types "Hello":**
   - Quill detects change
   - Emits `send-changes` with delta
   - Server broadcasts to room
   - User B receives `receive-changes`
   - User B's Quill updates → Sees "Hello" instantly!

6. **User B types " World":**
   - Same process in reverse
   - User A sees " World" appear
   - Document now shows "Hello World"

7. **Auto-save:**
   - Every 30 seconds, if modified
   - Emits `save-doc` with full content
   - Server saves to MongoDB
   - Toast notification: "Document saved"

8. **User A leaves:**
   - Clicks "Back" or closes tab
   - Emits `leaveRoom`
   - User B sees "User A left" notification

---

## 🛠️ Helper Functions Explained

### `auth.helper.js`
- **`login(user)`** - Sends POST request to login endpoint
- **`register(user)`** - Sends POST request to register endpoint
- **`setLocalStorageWithExpiry()`** - Saves data with expiration time
- **`getLocalStorageWithExpiry()`** - Retrieves data, checks if expired

### `doc.helper.js`
- **`createNewDoc(title, token)`** - Creates new document
- **`getAllLoggedInUserDocs(token)`** - Fetches all user's documents
- **`deleteTheDoc(docId, token)`** - Deletes a document
- **`addCollaboratorToDoc(docId, email, token)`** - Adds collaborator
- **`getAllCollaborators(docId, token)`** - Gets all collaborators

---

## 🎨 UI/UX Features

1. **Dark Mode**
   - Toggle button in navbar
   - Stored in SupplierContext
   - Applied to all components

2. **Loading States**
   - Loader component during API calls
   - Disabled buttons while loading
   - "Loading..." text

3. **Toast Notifications**
   - Success: Green toast
   - Error: Red toast
   - Info: Blue toast
   - Using `react-toastify`

4. **Responsive Design**
   - Bootstrap classes
   - Mobile-friendly
   - Collapsible sidebar

---

## 🔍 Important Code Patterns

### 1. **useEffect Hook Pattern**
```javascript
useEffect(() => {
  // Runs when component mounts or dependencies change
  fetchData();
  
  return () => {
    // Cleanup (runs when component unmounts)
    socket.off('event');
  };
}, [dependency1, dependency2]);
```

### 2. **Context Pattern**
```javascript
// Create context
const AuthContext = createContext();

// Provide context
<AuthContext.Provider value={{ auth, setAuth }}>
  {children}
</AuthContext.Provider>

// Use context
const { auth } = useAuth();
```

### 3. **Protected Route Pattern**
```javascript
const PrivateRoutes = () => {
  const token = getLocalStorageWithExpiry('auth')?.token;
  return token ? <Outlet /> : <Navigate to="/login" />;
};
```

### 4. **Socket.IO Pattern**
```javascript
// Emit (send)
socket.emit('event-name', data);

// Listen (receive)
socket.on('event-name', (data) => {
  // Handle data
});

// Cleanup
socket.off('event-name');
```

---

## 🐛 Common Issues & Solutions

1. **"Could not connect to MongoDB"**
   - Check MongoDB is running
   - Verify `MONGODB_URI` in `.env`

2. **"Token is not valid"**
   - Token expired (1 hour)
   - User needs to login again

3. **"Please verify your email first"**
   - Check email inbox
   - Click verification link

4. **Changes not syncing**
   - Check Socket.IO connection
   - Verify both users in same room
   - Check browser console for errors

---

## 📝 Summary

This project demonstrates:
- ✅ Full-stack MERN application
- ✅ User authentication with JWT
- ✅ Email verification
- ✅ Real-time collaboration with Socket.IO
- ✅ Rich text editing with Quill
- ✅ Protected routes
- ✅ RESTful API design
- ✅ Modern React patterns (hooks, context)
- ✅ Database relationships (references)
- ✅ Error handling
- ✅ Responsive UI

**Key Takeaway:** The magic happens when Socket.IO broadcasts changes to all users in a room, and Quill applies those changes in real-time!

---

## 🚀 Next Steps to Learn More

1. **Add features:**
   - Comments on documents
   - Version history
   - Export to PDF
   - Share via link

2. **Improve security:**
   - Rate limiting
   - Input validation
   - XSS protection

3. **Optimize:**
   - Pagination for documents
   - Image uploads
   - Search functionality

4. **Deploy:**
   - Deploy backend to Heroku/Railway
   - Deploy frontend to Vercel/Netlify
   - Use MongoDB Atlas (cloud database)

---

**Happy Coding! 🎉**


