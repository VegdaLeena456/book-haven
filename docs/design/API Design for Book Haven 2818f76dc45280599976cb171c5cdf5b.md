# API Design for Book Haven

# API Design for Book Haven

## Authentication

- **POST /api/auth/register**
    - **Purpose:** Register a new user.
    - **Input:** `{ email: string, password: string }`
    - **Output:** `{ message: "User created" }`
    - **Security:** Validate email format and password length (min 6 chars). Hash password with bcrypt before storing in MongoDB.
- **POST /api/auth/login**
    - **Purpose:** Login a user and return a JWT token.
    - **Input:** `{ email: string, password: string }`
    - **Output:** `{ token: "jwt-token" }`
    - **Security:** Verify password with bcrypt. Return JWT token for authenticated requests.

## Books

- **GET /api/books**
    - **Purpose:** List all books.
    - **Input:** None (optional: query params like `?title=bookName` for search later).
    - **Output:** `[{ id: string, title: string, author: string, price: number, description: string, imageUrl: string }, ...]`
    - **Security:** Public access (no token required).
- **GET /api/books/:id**
    - **Purpose:** Get details of a single book.
    - **Input:** URL param `id` (book ID).
    - **Output:** `{ id: string, title: string, author: string, price: number, description: string, imageUrl: string }`
    - **Security:** Public access.
- **POST /api/books**
    - **Purpose:** Add a new book (admin only).
    - **Input:** `{ title: string, author: string, price: number, description: string, imageUrl: string }`
    - **Output:** `{ message: "Book added", book: { id, title, author, price, description, imageUrl } }`
    - **Security:** Requires JWT token and admin role.
- **PUT /api/books/:id**
    - **Purpose:** Update a book’s details (admin only).
    - **Input:** URL param `id` (book ID), body `{ title: string, author: string, price: number, description: string, imageUrl: string }`
    - **Output:** `{ message: "Book updated", book: { id, title, author, price, description, imageUrl } }`
    - **Security:** Requires JWT token and admin role.
- **DELETE /api/books/:id**
    - **Purpose:** Delete a book (admin only).
    - **Input:** URL param `id` (book ID).
    - **Output:** `{ message: "Book deleted" }`
    - **Security:** Requires JWT token and admin role.

## Cart

- **GET /api/cart**
    - **Purpose:** View the user’s cart.
    - **Input:** None (user identified via JWT).
    - **Output:** `{ userId: string, items: [{ bookId: string, quantity: number }], total: number }`
    - **Security:** Requires JWT token (user-specific).
- **POST /api/cart**
    - **Purpose:** Add a book to the user’s cart.
    - **Input:** `{ bookId: string, quantity: number }`
    - **Output:** `{ message: "Item added to cart", cart: { userId, items, total } }`
    - **Security:** Requires JWT token.
- **DELETE /api/cart/:bookId**
    - **Purpose:** Remove a book from the user’s cart.
    - **Input:** URL param `bookId` (book ID to remove).
    - **Output:** `{ message: "Item removed from cart", cart: { userId, items, total } }`
    - **Security:** Requires JWT token.

## Orders

- **POST /api/orders**
    - **Purpose:** Create a new order from the cart.
    - **Input:** None (uses cart data; user identified via JWT).
    - **Output:** `{ message: "Order created", order: { id, userId, items: [{ bookId, quantity }], total, status } }`
    - **Security:** Requires JWT token. Simulate payment (no real payment gateway).
- **GET /api/orders**
    - **Purpose:** View the user’s order history.
    - **Input:** None (user identified via JWT).
    - **Output:** `[{ id: string, userId: string, items: [{ bookId, quantity }], total: number, status: string }, ...]`
    - **Security:** Requires JWT token.
- **GET /api/orders/all**
    - **Purpose:** View all orders (admin only).
    - **Input:** None.
    - **Output:** `[{ id: string, userId: string, items: [{ bookId, quantity }], total: number, status: string }, ...]`
    - **Security:** Requires JWT token and admin role.

## Search

- **GET /api/books/search**
    - **Purpose:** Search books by title or author (Should-have feature).
    - **Input:** Query params `?q=searchTerm` (e.g., `?q=Harry`).
    - **Output:** `[{ id, title, author, price, description, imageUrl }, ...]`
    - **Security:** Public access.

## Security & Scalability

- **Security:**
    - Hash passwords using bcrypt before storing in MongoDB.
    - Use JWT tokens for authentication (sent in Authorization header: `Bearer <token>`).
    - Validate all inputs (e.g., email format, positive price, valid bookId).
    - Restrict admin routes (/api/books POST, PUT, DELETE; /api/orders/all) to users with role: "admin".
- **Scalability:**
    - Add MongoDB indexes on `title` and `author` fields in Books collection for fast search.
    - Plan for caching (e.g., Redis) for frequently accessed data like book lists.
    - Use pagination for GET /api/books and GET /api/orders/all (e.g., `?page=1&limit=10`).