# LAB 2: Installing PostgreSQL and pgAdmin on macOS

## PostgreSQL server

I opened the Terminal and installed PostgreSQL with Homebrew using `brew install postgresql`. Then I started the server with `brew services start postgresql`.

Next, I connected to the database with `psql postgres`. On macOS the default user has the same name as my system username, so I set a password for it:

```sql
ALTER USER my_username WITH PASSWORD 'my_password';
```

After that, I exited the console with `\q`.

## pgAdmin

I installed pgAdmin with `brew install --cask pgadmin4` and opened it from Applications. I registered a new local server, filled in the Connection tab: host `localhost`, port `5432`, my username and the password I had set. After saving, the server appeared in the left panel.
