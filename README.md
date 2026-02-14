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

4.1 Check Ubuntu Version (optional but safe)
lsb_release -a

Codespaces typically run Ubuntu 24.04 (Noble). If it's different, adjust the repository URL accordingly.

💡 Option A: Install via Official Repository (Recommended if version matches)
Use this only if your Ubuntu version is 24.04 (Noble) or 22.04 (Jammy).

4.2 Remove any incorrect Swift repository (if you tried before)

sudo rm -f /etc/apt/sources.list.d/swift.list

4.3 Add the official Swift repository (for Ubuntu 24.04)

wget -q -O - https://swift.org/keys/automatic-signing-key-2024-05.asc | gpg --dearmor | sudo tee /usr/share/keyrings/swift.gpg
echo "deb [signed-by=/usr/share/keyrings/swift.gpg] https://download.swift.org/swift-6.0.2-release/ubuntu2404/swift-6.0.2-RELEASE /" | sudo tee /etc/apt/sources.list.d/swift.list

Note: If you have a different Ubuntu version, replace ubuntu2404 with ubuntu2204 (22.04) or ubuntu2004 (20.04).

4.4 Update package lists

sudo apt-get update

4.5 Install Swift

sudo apt-get install swiftlang

4.6 Verify installation

swift --version

You should see something like: Swift version 6.0.2 (swift-6.0.2-RELEASE).

⚠️ Note: You may see a harmless GPG warning about the Yarn repository – ignore it. If the Swift repository fails with "404 Not Found" or "Release file is missing", skip to Option B.

💡 Option B: Direct Download

Cleanup:

sudo rm -f /etc/apt/sources.list.d/swift.list

Check Ubuntu version:

lsb_release -a

run:

cd /workspaces/swift

Choose the URL based on your Ubuntu version:

⚠️ For Ubuntu 24.04:

wget https://download.swift.org/swift-6.0.2-release/ubuntu2404/swift-6.0.2-RELEASE/swift-6.0.2-RELEASE-ubuntu24.04.tar.gz
tar xzf swift-6.0.2-RELEASE-ubuntu24.04.tar.gz
echo 'export PATH=/workspaces/swift/swift-6.0.2-RELEASE-ubuntu24.04/usr/bin:$PATH' >> ~/.bashrc

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

💡 Option A: Using the VS Code File Explorer (Easiest)
In the left sidebar, click the Explorer icon (two sheets of paper).

Navigate to Sources/MySQLDemo/.

Right‑click inside the folder and select New File.

Name each file as follows (one by one):

configure.swift

routes.swift

ItemManager.swift

User.swift

AuthMiddleware.swift

Double‑click each file to open it, then copy the code from the video (or from the provided code snippets) and paste it in.

Save each file with Ctrl+S.

💡 Option B: Using the Terminal (if you prefer)

Navigate to the source folder:

cd Sources/MySQLDemo
Then create each empty file with touch:

touch configure.swift routes.swift ItemManager.swift User.swift AuthMiddleware.swift

Now open each file in the editor (e.g., nano configure.swift) and paste the code from the video.

💡 Video Note: The exact code for each file will be shown in the video. You can pause and copy it directly. Alternatively, the code is available in the video description or a linked gist.

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

13. Commit and Push to GitHub
Before committing, create a .gitignore file in the repository root (/workspaces/swift) to exclude build artifacts:

.build/
Packages/
.swiftpm/
*.tar.gz

Then:

cd /workspaces/swift
git add .
git commit -m "Add complete Swift MySQL web app"
git push

14. Once everything is done and u need to run the program
In your terminal, run:

docker ps | grep mysql-demo

If you see no output, the container is not running. Start it:

docker start mysql-demo

Run this to test the connection:

docker exec mysql-demo mysqladmin ping -u root -psecret

You should see mysqld is alive. If not, wait a bit and retry.

Connect to MySQL:

docker exec -it mysql-demo mysql -u root -psecret

Run it:

cd /workspaces/swift/MySQLDemo

swift run

MySQL is used with Swift with these accomplishments:

✅ Successfully connected Swift to MySQL using Vapor's MySQLKit

✅ Created and managed database tables (users and items)

✅ Performed full CRUD operations (add, edit, delete, search)

✅ Handled user authentication with MySQL storing usernames and passwords

✅ The App is running right now with MySQL as its database
