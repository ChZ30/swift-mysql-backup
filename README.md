# swift

Prerequisites
A GitHub account to use Codespaces (recommended para magamit sa Windows)

Optional: Swift 6.0+ and MySQL 8.0 installed if running locally (pang linux / MacOS)



For Users Using Codespaces

Start the MySQL container (if not already running):
docker start mysql-demo

(If the container doesn’t exist, create it with the docker run command from the README.)

Connect to the MySQL prompt inside the container:
docker exec -it mysql-demo mysql -u root -p
Enter password "secret"

Now paste the SQL:

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

After each command, you’ll see Query OK.
Type exit to leave MySQL.



📌 For Users Running Locally (with MySQL installed)

Open a terminal and start the MySQL command‑line client:
mysql -u root -p
(Enter your MySQL root password.)

Paste the SQL (same block as above).

Type exit when done.


💡 Important Note
The app will automatically create the users and items tables the first time you sign up or add an item (if they don’t already exist). So running the SQL manually is optional, but it’s included to give you full control and to show the schema.

If you ever need to reset the database, you can drop the tables and let the app recreate them, or re‑run the SQL.


Default Login
After setup, you can log in with:

Username: admin

Password: secret

You can also create new accounts via the Sign up link.
