# swift

1. Prerequisites
A GitHub account to use Codespaces (recommended para magamit sa Windows)

Optional: Swift 6.0+ and MySQL 8.0 installed if running locally (pang linux / MacOS)

2. Create a GitHub Repository
Log in to GitHub.

Click the + icon (top right) → New repository.

Name it "swift", make it Public (or Private if you prefer).

Do not initialize with a README (you can add one later).

Click Create repository.

3. Launch a Codespace
On the repository page, click the green Code button.

Select the Codespaces tab → Create codespace on main.

Wait a minute for the environment to build. You’ll get a full VS Code in your browser.

4. Install Swift in the Codespace
Open the terminal in VS Code (Ctrl + `).

Check Ubuntu Version
lsb_release -a

Cleanup:

sudo rm -f /etc/apt/sources.list.d/swift.list

Run:

⚠️ For codes that include “/workspaces/swift/”, CHANGE "swift" into your REPOSITORY NAME ⚠️

cd /workspaces/swift

Choose the URL based on your Ubuntu version:

⚠️ For Ubuntu 24.04:

wget https://download.swift.org/swift-6.0.2-release/ubuntu2404/swift-6.0.2-RELEASE/swift-6.0.2-RELEASE-ubuntu24.04.tar.gz
tar xzf swift-6.0.2-RELEASE-ubuntu24.04.tar.gz
echo 'export PATH=/workspaces/swift/swift-6.0.2-RELEASE-ubuntu24.04/usr/bin:$PATH' >> ~/.bashrc

⚠️ For codes that include “/workspaces/swift/”, CHANGE "swift" into your REPOSITORY NAME ⚠️

⚠️ For Ubuntu 22.04:

wget https://download.swift.org/swift-6.0.2-release/ubuntu2204/swift-6.0.2-RELEASE/swift-6.0.2-RELEASE-ubuntu22.04.tar.gz
tar xzf swift-6.0.2-RELEASE-ubuntu22.04.tar.gz
echo 'export PATH=/workspaces/swift/swift-6.0.2-RELEASE-ubuntu22.04/usr/bin:$PATH' >> ~/.bashrc

Then reload your shell configuration:

source ~/.bashrc

Verify Swift Installation:

swift --version

You should see output like Swift version 6.0.2 (swift-6.0.2-RELEASE).

5. Install MySQL Client Libraries

sudo apt-get install -y libmysqlclient-dev

These are needed for Swift packages that interact with MySQL.

6. Create the Swift Project

cd ~
mkdir MySQLDemo
cd MySQLDemo
swift package init --type executable

7. Edit Package.swift to Add Dependencies
Open Package.swift and replace its content with the final version (provided in the video). It will include:

vapor – web framework

mysql-kit – MySQL driver

leaf – templating engine

Code for “Package.swift”:

cat > Package.swift << 'EOF'
// swift-tools-version:5.7
import PackageDescription

let package = Package(
    name: "MySQLDemo",
    platforms: [.macOS(.v12)],
    dependencies: [
        .package(url: "https://github.com/vapor/vapor.git", from: "4.0.0"),
        .package(url: "https://github.com/vapor/mysql-kit.git", from: "4.0.0"),
        .package(url: "https://github.com/vapor/leaf.git", from: "4.0.0")
    ],
    targets: [
        .executableTarget(
            name: "MySQLDemo",
            dependencies: [
                .product(name: "Vapor", package: "vapor"),
                .product(name: "MySQLKit", package: "mysql-kit"),
                .product(name: "Leaf", package: "leaf")
            ],
            path: "Sources/MySQLDemo"
        )
    ]
)
EOF

After saving, run:

swift package resolve

Note that this will take a couple of minutes.

8. Set Up MySQL Using Docker

8.1 Start the MySQL Container
If the container doesn't exist yet, create it with:

docker run --name mysql-demo -e MYSQL_ROOT_PASSWORD=secret -e MYSQL_DATABASE=testdb -p 3306:3306 -d mysql:8.0

Wait about 10–15 seconds for MySQL to fully initialize.

If the container already exists (e.g., from a previous session), just start it:

docker start mysql-demo

8.2 Connect to MySQL and Create Tables

WAIT AROUND 1 MINUTE OR SO, then run:

docker exec -it mysql-demo mysql -u root -p

Enter the password: secret 💡(even if nothing appears to be typing, JUST TYPE IT)

Once inside the MySQL prompt (mysql>), paste the following SQL:

CREATE DATABASE IF NOT EXISTS testdb;
USE testdb;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL
);

INSERT INTO users (username, password) VALUES ('admin', 'secret');

CREATE TABLE items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

Each command should show Query OK. After finishing, type:

exit

💡 Note: The application can create these tables automatically the first time you sign up or add an item, but running the SQL manually ensures everything is set up correctly and gives you full control.

9. Create the Swift Source Files
After initializing the project (step 6), you'll have a Sources/MySQLDemo/ folder containing main.swift. Now add the remaining files:

Navigate to the source folder:

cd Sources/MySQLDemo

If you already have a project but the Sources/MySQLDemo folder is missing:

cd ~/MySQLDemo
mkdir -p Sources/MySQLDemo

Then create each empty file with touch:

touch configure.swift routes.swift ItemManager.swift User.swift AuthMiddleware.swift

Now open each file in the editor (e.g., nano configure.swift) and paste the code from the video.

main.swift: 

cat > Sources/MySQLDemo/main.swift << 'EOF'
import Vapor

var env = try Environment.detect()
try LoggingSystem.bootstrap(from: &env)
let app = Application(env)
defer { app.shutdown() }

try configure(app)
try routes(app)
try app.run()
EOF

Configure.swift:

cat > Sources/MySQLDemo/configure.swift << 'EOF'
import Vapor
import MySQLKit
import Leaf

extension Application {
    private struct ItemManagerKey: StorageKey {
        typealias Value = ItemManager
    }
    var itemManager: ItemManager {
        get {
            guard let manager = self.storage[ItemManagerKey.self] else {
                fatalError("ItemManager not configured")
            }
            return manager
        }
        set { self.storage[ItemManagerKey.self] = newValue }
    }

    private struct UserAuthKey: StorageKey {
        typealias Value = UserAuthenticator
    }
    var userAuth: UserAuthenticator {
        get {
            guard let auth = self.storage[UserAuthKey.self] else {
                fatalError("UserAuthenticator not configured")
            }
            return auth
        }
        set { self.storage[UserAuthKey.self] = newValue }
    }

    struct ConnectionPoolStorageKey: StorageKey {
        typealias Value = EventLoopGroupConnectionPool<MySQLConnectionSource>
    }
}

func configure(_ app: Application) throws {
    app.views.use(.leaf)
    app.middleware.use(app.sessions.middleware)
    app.sessions.use(.memory)

    let configuration = MySQLConfiguration(
        hostname: Environment.get("DB_HOST") ?? "127.0.0.1",
        port: Environment.get("DB_PORT").flatMap(Int.init) ?? 3306,
        username: Environment.get("DB_USER") ?? "root",
        password: Environment.get("DB_PASSWORD") ?? "secret",
        database: Environment.get("DB_NAME") ?? "testdb",
        tlsConfiguration: nil
    )

    let pools = EventLoopGroupConnectionPool(
        source: MySQLConnectionSource(configuration: configuration),
        on: app.eventLoopGroup
    )

    app.storage[Application.ConnectionPoolStorageKey.self] = pools
    app.lifecycle.use(ConnectionPoolCleanup())

    let db = pools.database(logger: Logger(label: "mysql"))
    app.itemManager = ItemManager(db: db)
    app.userAuth = UserAuthenticator(db: db)
}

struct ConnectionPoolCleanup: LifecycleHandler {
    func shutdown(_ application: Application) throws {
        try application.storage[Application.ConnectionPoolStorageKey.self]?.syncShutdownGracefully()
    }
}
EOF

routes.swift:

cat > Sources/MySQLDemo/routes.swift << 'EOF'
import Vapor

struct IndexContext: Encodable {
    let items: [Item]
    let searchTerm: String
    let username: String?
}

struct EditContext: Encodable {
    let item: Item
    let username: String?
}

func routes(_ app: Application) throws {
    let manager = app.itemManager
    let auth = app.userAuth

    // Login
    app.get("login") { req async throws -> View in
        let error = req.query["error"] ?? ""
        let registered = req.query["registered"] ?? ""
        return try await req.view.render("login", ["error": error, "registered": registered])
    }

    app.post("login") { req async throws -> Response in
        struct LoginData: Content {
            var username: String
            var password: String
        }
        let data = try req.content.decode(LoginData.self)
        let user = try await auth.authenticate(username: data.username, password: data.password).get()
        if let user = user {
            req.session.data["userId"] = String(user.id)
            req.session.data["username"] = user.username
            return req.redirect(to: "/")
        } else {
            return req.redirect(to: "/login?error=Invalid%20username%20or%20password")
        }
    }

    // Signup
    app.get("signup") { req async throws -> View in
        let error = req.query["error"] ?? ""
        return try await req.view.render("signup", ["error": error])
    }

    app.post("signup") { req async throws -> Response in
        struct SignupData: Content {
            var username: String
            var password: String
        }
        let data = try req.content.decode(SignupData.self)
        let newUser = try await auth.createUser(username: data.username, password: data.password).get()
        if let user = newUser {
            req.session.data["userId"] = String(user.id)
            req.session.data["username"] = user.username
            return req.redirect(to: "/")
        } else {
            return req.redirect(to: "/signup?error=Username%20already%20taken")
        }
    }

    // Logout
    app.get("logout") { req -> Response in
        req.session.destroy()
        return req.redirect(to: "/login")
    }

    // Protected routes
    let protected = app.grouped(AuthMiddleware())

    protected.get { req async throws -> View in
        let items = try await manager.searchItems(term: "").get()
        let username = req.session.data["username"]
        let context = IndexContext(items: items, searchTerm: "", username: username)
        return try await req.view.render("index", context)
    }

    protected.get("search") { req async throws -> View in
        let term = req.query["term"] ?? ""
        let items = try await manager.searchItems(term: term).get()
        let username = req.session.data["username"]
        let context = IndexContext(items: items, searchTerm: term, username: username)
        return try await req.view.render("index", context)
    }

    protected.get("add") { req async throws -> View in
        let username = req.session.data["username"]
        return try await req.view.render("add", ["username": username])
    }

    protected.post("add") { req async throws -> Response in
        struct AddData: Content {
            var name: String
            var description: String?
        }
        let data = try req.content.decode(AddData.self)
        try await manager.addItem(name: data.name, description: data.description).get()
        return req.redirect(to: "/")
    }

    protected.get("edit", ":id") { req async throws -> View in
        guard let id = req.parameters.get("id", as: Int.self) else {
            throw Abort(.badRequest)
        }
        guard let item = try await manager.getItem(id: id).get() else {
            throw Abort(.notFound)
        }
        let username = req.session.data["username"]
        let context = EditContext(item: item, username: username)
        return try await req.view.render("edit", context)
    }

    protected.post("edit", ":id") { req async throws -> Response in
        struct EditData: Content {
            var name: String?
            var description: String?
        }
        guard let id = req.parameters.get("id", as: Int.self) else {
            throw Abort(.badRequest)
        }
        let data = try req.content.decode(EditData.self)
        try await manager.updateItem(id: id, name: data.name, description: data.description).get()
        return req.redirect(to: "/")
    }

    protected.post("delete", ":id") { req async throws -> Response in
        guard let id = req.parameters.get("id", as: Int.self) else {
            throw Abort(.badRequest)
        }
        try await manager.deleteItem(id: id).get()
        return req.redirect(to: "/")
    }
}
EOF

ItemManager.swift:

cat > Sources/MySQLDemo/ItemManager.swift << 'EOF'
import MySQLKit
import NIOCore

typealias Database = MySQLDatabase

struct Item: Codable {
    let id: Int
    let name: String
    let description: String?
}

struct ItemManager {
    let db: Database

    func addItem(name: String, description: String?) -> EventLoopFuture<Void> {
        let descValue = description != nil ? "'\(description!)'" : "NULL"
        let insert = """
            INSERT INTO items (name, description)
            VALUES ('\(name)', \(descValue))
        """
        return db.simpleQuery(insert).map { _ in }
    }

    func updateItem(id: Int, name: String?, description: String?) -> EventLoopFuture<Void> {
        var updates: [String] = []
        if let newName = name, !newName.isEmpty {
            updates.append("name = '\(newName)'")
        }
        if let newDesc = description {
            updates.append("description = '\(newDesc)'")
        }
        guard !updates.isEmpty else {
            return db.eventLoop.makeSucceededFuture(())
        }
        let setClause = updates.joined(separator: ", ")
        let query = "UPDATE items SET \(setClause) WHERE id = \(id)"
        return db.simpleQuery(query).map { _ in }
    }

    func deleteItem(id: Int) -> EventLoopFuture<Void> {
        let query = "DELETE FROM items WHERE id = \(id)"
        return db.simpleQuery(query).map { _ in }
    }

    func getItem(id: Int) -> EventLoopFuture<Item?> {
        let query = "SELECT id, name, description FROM items WHERE id = \(id)"
        return db.simpleQuery(query).map { rows in
            guard let row = rows.first,
                  let id = row.column("id")?.int,
                  let name = row.column("name")?.string else { return nil }
            let description = row.column("description")?.string
            return Item(id: id, name: name, description: description)
        }
    }

    func searchItems(term: String) -> EventLoopFuture<[Item]> {
        let query = """
            SELECT id, name, description FROM items
            WHERE name LIKE '%\(term)%' OR description LIKE '%\(term)%'
        """
        return db.simpleQuery(query).map { rows in
            rows.compactMap { row in
                guard let id = row.column("id")?.int,
                      let name = row.column("name")?.string else { return nil }
                let description = row.column("description")?.string
                return Item(id: id, name: name, description: description)
            }
        }
    }
}
EOF

User.swift:

cat > Sources/MySQLDemo/User.swift << 'EOF'
import Vapor
import MySQLKit

struct User: Content {
    let id: Int
    let username: String
}

struct UserAuthenticator {
    let db: MySQLDatabase

    func authenticate(username: String, password: String) -> EventLoopFuture<User?> {
        let query = "SELECT id, username FROM users WHERE username = '\(username)' AND password = '\(password)'"
        return db.simpleQuery(query).map { rows in
            guard let row = rows.first,
                  let id = row.column("id")?.int,
                  let username = row.column("username")?.string else {
                return nil
            }
            return User(id: id, username: username)
        }
    }

    func createUser(username: String, password: String) -> EventLoopFuture<User?> {
        let checkQuery = "SELECT id FROM users WHERE username = '\(username)'"
        return db.simpleQuery(checkQuery).flatMap { rows in
            if !rows.isEmpty {
                return db.eventLoop.makeSucceededFuture(nil)
            }
            let insertQuery = "INSERT INTO users (username, password) VALUES ('\(username)', '\(password)')"
            return db.simpleQuery(insertQuery).flatMap { _ in
                let selectQuery = "SELECT id, username FROM users WHERE username = '\(username)'"
                return db.simpleQuery(selectQuery).map { rows in
                    guard let row = rows.first,
                          let id = row.column("id")?.int,
                          let username = row.column("username")?.string else {
                        return nil
                    }
                    return User(id: id, username: username)
                }
            }
        }
    }
}
EOF

AuthMiddleware.swift:

cat > Sources/MySQLDemo/AuthMiddleware.swift << 'EOF'
import Vapor

struct AuthMiddleware: Middleware {
    func respond(to request: Request, chainingTo next: Responder) -> EventLoopFuture<Response> {
        if request.session.data["userId"] != nil {
            return next.respond(to: request)
        } else {
            return request.eventLoop.makeSucceededFuture(
                request.redirect(to: "/login")
            )
        }
    }
}
EOF

💡 Video Note: The exact code for each file will be shown in the video. You can pause and copy it directly. Alternatively, the code is available in the video description or a linked gist.

10. Create Leaf Templates
Create the directory:

mkdir -p Resources/Views

login.leaf:

cat > Resources/Views/login.leaf << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Login</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .button {
            display: inline-block;
            padding: 6px 12px;
            margin: 2px;
            background-color: #007bff;
            color: white;
            text-decoration: none;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 14px;
        }
        .button:hover { background-color: #0056b3; }
        input { margin-bottom: 10px; }
        .error { color: red; }
        .success { color: green; }
    </style>
</head>
<body>
    <h1>Login</h1>
    #if(error):
        <p class="error">#(error)</p>
    #endif
    #if(registered):
        <p class="success">#(registered)</p>
    #endif
    <form action="/login" method="post">
        <label>Username:</label><br>
        <input type="text" name="username" required><br>
        <label>Password:</label><br>
        <input type="password" name="password" required><br>
        <button type="submit" class="button">Login</button>
    </form>
    <p>Don't have an account? <a href="/signup" class="button">Sign up</a></p>
</body>
</html>
EOF

signup.leaf:

cat > Resources/Views/signup.leaf << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Sign Up</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .button {
            display: inline-block;
            padding: 6px 12px;
            margin: 2px;
            background-color: #007bff;
            color: white;
            text-decoration: none;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 14px;
        }
        .button:hover { background-color: #0056b3; }
        .button.cancel { background-color: #6c757d; }
        .button.cancel:hover { background-color: #5a6268; }
        input { margin-bottom: 10px; }
        .error { color: red; }
    </style>
</head>
<body>
    <h1>Create an Account</h1>
    #if(error):
        <p class="error">#(error)</p>
    #endif
    <form action="/signup" method="post">
        <label>Username:</label><br>
        <input type="text" name="username" required><br>
        <label>Password:</label><br>
        <input type="password" name="password" required><br>
        <button type="submit" class="button">Sign Up</button>
    </form>
    <p>Already have an account? <a href="/login" class="button cancel">Log in</a></p>
</body>
</html>
EOF

index.leaf:

cat > Resources/Views/index.leaf << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Item Management</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #f2f2f2; }
        .button {
            display: inline-block;
            padding: 6px 12px;
            margin: 2px;
            background-color: #007bff;
            color: white;
            text-decoration: none;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 14px;
        }
        .button:hover { background-color: #0056b3; }
        .button.delete { background-color: #dc3545; }
        .button.delete:hover { background-color: #b02a37; }
        .button.search { background-color: #28a745; }
        .button.search:hover { background-color: #1e7e34; }
        .button.add { background-color: #28a745; }
        .button.add:hover { background-color: #1e7e34; }
        form { display: inline; }
        .search-row {
            display: flex;
            align-items: center;
            gap: 8px;
            margin-bottom: 20px;
            flex-wrap: wrap;
        }
        .search-row input[type="text"] {
            padding: 6px 8px;
            font-size: 14px;
            border: 1px solid #ccc;
            border-radius: 4px;
            width: 200px;
        }
        .search-row form {
            display: flex;
            align-items: center;
            gap: 8px;
            margin: 0;
        }
    </style>
</head>
<body>
    #if(username):
        <p>Logged in as #(username) | <a href="/logout" class="button">Logout</a></p>
    #endif
    <h1>Item Management</h1>

    <div class="search-row">
        <form action="/search" method="get">
            <input type="text" name="term" placeholder="Search..." value="#(searchTerm)">
            <button type="submit" class="button search">Search</button>
            <a href="/" class="button">Clear</a>
        </form>
        <a href="/add" class="button add">Add Item</a>
    </div>

    <table>
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Description</th>
            <th>Actions</th>
        </tr>
        #for(item in items):
        <tr>
            <td>#(item.id)</td>
            <td>#(item.name)</td>
            <td>
                #if(item.description):
                    #(item.description)
                #else:
                    —
                #endif
            </td>
            <td>
                <a href="/edit/#(item.id)" class="button">Edit</a>
                <form action="/delete/#(item.id)" method="post" style="display:inline;">
                    <button type="submit" class="button delete" onclick="return confirm('Delete this item?')">Delete</button>
                </form>
            </td>
        </tr>
        #endfor
    </table>
</body>
</html>
EOF

add.leaf:

cat > Resources/Views/add.leaf << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Add Item</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .button {
            display: inline-block;
            padding: 6px 12px;
            margin: 2px;
            background-color: #007bff;
            color: white;
            text-decoration: none;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 14px;
        }
        .button:hover { background-color: #0056b3; }
        .button.cancel { background-color: #6c757d; }
        .button.cancel:hover { background-color: #5a6268; }
        input, textarea { margin-bottom: 10px; }
    </style>
</head>
<body>
    #if(username):
        <p>Logged in as #(username) | <a href="/logout" class="button">Logout</a></p>
    #endif
    <h1>Add New Item</h1>
    <form action="/add" method="post">
        <label>Name:</label><br>
        <input type="text" name="name" required><br>
        <label>Description (optional):</label><br>
        <textarea name="description"></textarea><br>
        <button type="submit" class="button">Add Item</button>
        <a href="/" class="button cancel">Cancel</a>
    </form>
</body>
</html>
EOF

edit.leaf:

cat > Resources/Views/edit.leaf << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Edit Item</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .button {
            display: inline-block;
            padding: 6px 12px;
            margin: 2px;
            background-color: #007bff;
            color: white;
            text-decoration: none;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 14px;
        }
        .button:hover { background-color: #0056b3; }
        .button.cancel { background-color: #6c757d; }
        .button.cancel:hover { background-color: #5a6268; }
        input, textarea { margin-bottom: 10px; width: 300px; }
    </style>
</head>
<body>
    #if(username):
        <p>Logged in as #(username) | <a href="/logout" class="button">Logout</a></p>
    #endif
    <h1>Edit Item</h1>
    <form action="/edit/#(item.id)" method="post">
        <label>Name:</label><br>
        <input type="text" name="name" value="#(item.name)"><br>
        <label>Description:</label><br>
        <textarea name="description">#(item.description)</textarea><br>
        <button type="submit" class="button">Update Item</button>
        <a href="/" class="button cancel">Cancel</a>
    </form>
</body>
</html>
EOF

11. Build and Run the App

swift build
swift run

SUPER TAGAL nito kaya abang muna

The server starts on port 8080. In Codespaces, a pop‑up will appear – click Open in Browser to see the app.

12. Test the Application
Login: Use admin / secret (or sign up a new user).

Add items: Fill in name and description.

Edit items: Click Edit, modify, and save.

Delete items: Click Delete and confirm.

Search: Enter a term in the search box.

Logout: Click Logout.

Error messages (wrong login, duplicate signup) appear on the page.

Default Login
After setup, you can log in with:

Username: admin

Password: secret

You can also create new accounts via the Sign up link.

13. Commit and Push to GitHub

⚠️ For codes that include “/workspaces/swift/”, CHANGE "swift" into your REPOSITORY NAME ⚠️

Move the project into the repository:

mv ~/MySQLDemo /workspaces/swift/

Go to the repository root (if you're not already there):

cd /workspaces/swift

Create or edit .gitignore:

nano .gitignore

Paste the following content:

.build/
Packages/
.swiftpm/
*.tar.gz

Press (Ctrl + S) and  (Ctrl + X)

Add your project folder (and any other files you want to push):

git add MySQLDemo

Commit and push:

git commit -m "Add complete Swift MySQL web app"
git push

If this is the first push from this branch, you might need:

git push -u origin main

14. Once everything is done and you want to run the program (if codespace has restarted)

In your terminal, run:

docker ps | grep mysql-demo

If you see no output, the container is not running. Start it:

docker start mysql-demo

Run this to test the connection:

docker exec mysql-demo mysqladmin ping -u root -psecret

You should see “mysqld is alive”. If not, WAIT A BIT and retry.

Run it:

cd /workspaces/swift/MySQLDemo

swift run

MySQL is used with Swift with these accomplishments:

✅ Successfully connected Swift to MySQL using Vapor's MySQLKit

✅ Created and managed database tables (users and items)

✅ Performed full CRUD operations (add, edit, delete, search)

✅ Handled user authentication with MySQL storing usernames and passwords

✅ The App is running right now with MySQL as its database

