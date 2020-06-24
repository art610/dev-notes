# MiniDevLab on Ubuntu

## PostgreSQL 12.3 installation on Linux Ubuntu 18.04

```bash
# Create the file repository configuration:
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Import the repository signing key:
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Update the package lists:
sudo apt-get update

# Install the latest version of PostgreSQL
sudo apt-get install postgresql-12

# Login like postgres
su - postgres
# Use psql - check version
psql
SELECT version();

# Create user and DB
CREATE DATABASE <dbname> with encoding='UNICODE';
CREATE USER <username> with password '<userpassword>';
GRANT ALL PRIVILEGES ON DATABASE <dbname> TO <username>;

# exit
\q
exit

# Install additional packages
sudo apt-get install libpq-dev postgresql-contrib
```

## WikiJS installation

Create postgres user for wikijs:
```
su - postgres
psql
CREATE DATABASE wikijsdb with encoding='UNICODE';
CREATE USER wikijsdbuser with password 'naI@93*w';
GRANT ALL PRIVILEGES ON DATABASE wikijsdb TO wikijsdbuser;
```

Install wikiJS
```
# Download the latest version of Wiki.js
wget https://github.com/Requarks/wiki/releases/download/2.4.107/wiki-js.tar.gz
# Extract the package to the final destination of your choice
mkdir wiki
tar xzf wiki-js.tar.gz -C ./wiki
cd ./wiki
# Rename the sample config file to config.yml
mv config.sample.yml config.yml
```

Edit the config file and fill in your database and port settings
`nano config.yml`
Port:
`port: 3000`

Database
```
db:
  type: postgres
  host: localhost
  port: 5432
  user: wikijsdbuser
  pass: naI@93*w
  db: wikijsdb
```

Offline Mode
```
offline: true
```

 HTTPS
 You need both the private key (key) and certificate (cert) in PEM format
 ```
 ssl:
  enabled: true
  port: 3443
  provider: custom

  format: pem
  key: path/to/key.pem
  cert: path/to/cert.pem
  passphrase: null
  dhparam: null
 ```
 
 It's also possible to use a PFX (pem) formatted certificate instead:
 ```
 ssl:
  enabled: true
  port: 3443
  provider: custom

  format: pfx
  pfx: path/to/cert.pfx
  passphrase: null
  dhparam: null
 ```

Let's Encrypt allows for free, automated and auto-renewing SSL certificates for your wiki
```
ssl:
  enabled: true
  port: 3443
  provider: letsencrypt

  domain: wiki.yourdomain.com
  subscriberEmail: admin@example.com
```

Once your HTTPS is up and working correctly, you can enable HTTP to HTTPS redirection under the Administration Area > SSL.

More configs on https://docs.requarks.io/install/config

Run wikijs:
```
node server
```

