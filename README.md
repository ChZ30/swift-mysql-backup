# swift

1. Prerequisites
A GitHub account to use Codespaces (recommended para magamit sa Windows)

Optional: Swift 6.0+ and MySQL 8.0 installed if running locally (pang linux / MacOS)

2. Create a GitHub Repository
Log in to GitHub.

Click the + icon (top right) → New repository.

Name it (e.g., swift-mysql-app), make it Public (or Private if you prefer).

Do not initialize with a README (you can add one later).

Click Create repository.

3. Launch a Codespace
On the repository page, click the green Code button.

Select the Codespaces tab → Create codespace on main.

Wait a minute for the environment to build. You’ll get a full VS Code in your browser.

4. Install Swift in the Codespace
Open the terminal in VS Code (Ctrl + `).

Run these commands to add the official Swift repository terminal and install Swift:

wget -q -O - https://swift.org/keys/automatic-signing-key-2024-05.asc | gpg --dearmor | sudo tee /usr/share/keyrings/swift.gpg
echo "deb [signed-by=/usr/share/keyrings/swift.gpg] https://download.swift.org/swift-6.0.2-release/ubuntu2404/swift-6.0.2-RELEASE /" | sudo tee /etc/apt/sources.list.d/swift.list
sudo apt-get update
sudo apt-get install swiftlang

Verify with:
swift --version

5. Install MySQL Client Libraries

sudo apt-get install -y libmysqlclient-dev

6. Create the Swift Project

cd ~
mkdir MySQLDemo
cd MySQLDemo
swift package init --type executable

7. Edit Package.swift to Add Dependencies
Open Package.swift and replace its content with the final version (shown in the video). It will include:

vapor – web framework

mysql-kit – MySQL driver

leaf – templating engine

After saving, run:

swift package resolve

8. Set Up MySQL Using Docker

8.1 Start the MySQL Container
If the container doesn't exist yet, create it with:

docker run --name mysql-demo -e MYSQL_ROOT_PASSWORD=secret -e MYSQL_DATABASE=testdb -p 3306:3306 -d mysql:8.0

Wait about 10–15 seconds for MySQL to fully initialize.

If the container already exists (e.g., from a previous session), just start it:

docker start mysql-demo

8.2 Connect to MySQL and Create Tables
Now connect to the MySQL prompt inside the container:

docker exec -it mysql-demo mysql -u root -p

Enter the password: secret

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
Inside Sources/MySQLDemo/, create the following files (exact code shown in video):

configure.swift – sets up Leaf, sessions, and MySQL connection pool.

routes.swift – defines all HTTP endpoints.

ItemManager.swift – contains Item model and database operations.

User.swift – contains User model and authentication logic.

AuthMiddleware.swift – protects routes that require login.

main.swift (already exists) – starts the Vapor application.

10. Create Leaf Templates
Create the directory:

mkdir -p Resources/Views

11. Build and Run the App

swift build
swift run

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
