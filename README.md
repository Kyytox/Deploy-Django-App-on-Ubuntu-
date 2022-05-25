# Deploy-Django-App-on-Ubuntu-
Hello,
<br>
sorry for the terms used and the possible useless commands
<br>
<br>
<br>


# Web hosting 
Find host for u app
<br>
Create a server Ubuntu 20.04
<br>
<br>
<br>


# Server
### First connection

Open terminal Ubuntu or GitBash
```
ssh root@MyIP
```
<br>


### Update server
Update and upgrade server 
```
apt update && apt upgrade 
```
<br>

Install python3 and PostgreSql 
```
sudo apt-get install python3-pip python3-dev libpq-dev python3-pip python-distribute python-dev postgresql postgresql-contrib
```
```
sudo apt-get install python3-tk
```
<br>

Install pack psycopg2 for PostgreSql
```
pip3 install psycopg2
```
<br>

Install django framewwork
```
pip3 install django
```
<br>


### Define your Hostname
```
hostnamectl set-hostname MyHOSTNAME 
```
<br>

Update file hosts
```
nano /etc/hosts
```
<br>

Ajouter a la suite des IP déjà existante 
```
MyIP myhostname
```
<br>


### Ajouter un utilisateur
Define your user for avoid use admin user, for best security 
```
adduser MyUser
```
<br>

Add user to sudo group
```
adduser MyUser sudo     
```
<br>

Exit for reconnect with the new user 
```
exit
```
<br>

Reconnect with new user
```
ssh MyUser@MyIP
```
<br>



### Install Firewall 


```
sudo apt install ufw
```

```
sudo ufw default allow outgoing 
```

```
sudo ufw default deny incoming 
```

```
sudo ufw allow ssh 
```

```
sudo ufw allow 8000
```

```
sudo ufw allow http 
```

```
sudo ufw enable 
```

```
sudo ufw status 
```

```
exit
```
<br>
<br>
<br>


# Upload App Django
### Transfer App
Transfer app on server in folder /home/`MyUser` 
```
scp -r MyApp MyUser@MyIP:~/
```
<br>

Connect with user
```
ssh MyUser@MyIP
```
<br>
<br>


### Install app and dependencies 

Install environnment python
```
sudo apt install python3-venv
```
<br>

Go to folder App
```
cd MONAPP
```
<br>

Create virtual environnment in App
```
python3 -m venv venv
```
<br>

Active virtual environnment
```
source venv/bin/activate
```
<br>

Install dependencies
```
sudo pip install -r requirements.txt 
```
<br>

Install django Framework
```
pip3 install django
```
<br>
<br>



### Configurer son app 

Update file settings
```
sudo nano MONAPP/settings.py         
```
<br>


Add IP in param ALLOWED_HOSTS:
```
ALLOWED_HOSTS = ['MyIP']
```
<br>


Add STATIC_ROOT for static files
```
STATIC_ROOT = os.path.join(BASE_DIR, 'nameFolderProject/staticfiles')
```
<br>


### Transfer static files in folde define in settings.py, param= STATIC_ROOT
```
python manage.py collectstatic 
```
<br>
<br>
<br>




# POSTGRE SQL

Delete all migrations to avoid data repatriation failed
<br>

### create BD 

Connect with admin user at BD PostgreSQL  
```
sudo -u postgres psql
```
<br>

Create BD 
```
CREATE DATABASE nameBD;
```
<br>

Create user 
```
CREATE USER nomUser WITH PASSWORD 'Password';
```
<br>

Update Role
```
ALTER ROLE nomUser SET client_encoding TO 'utf8';
```

```
ALTER ROLE nomUser SET default_transaction_isolation TO 'read committed';
```

```
ALTER ROLE nomUser SET timezone TO 'Europe/Paris';
```
<br>

Add all privileges to the user for the BD 
```
GRANT ALL PRIVILEGES ON DATABASE nomBD TO nomUser;
```
<br>

Exit
```
\q
```
<br>


### Update BD param in settings.py

Update settings.py
```
sudo nano MyApp/settings.py
```
<br>


Add this code in DATABASE param	
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'nameBD',
        'USER': 'nameUser',
        'PASSWORD': 'Password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```
<br>


### Launch migrations

```
python manage.py makemigrations
```

```
python manage.py migrate
```
<br>


Create super user for administrate BD directly in web page xxxxxx.fr/admin
```
python manage.py createsuperuser
```
<br>

Launch App
```
python manage.py runserver 0.0.0.0:8000
```
<br> 
In web write `MyIP:8000`

<br>
<br>
<br>


# NGINX
### Install 
```
sudo apt-get install nginx
```
<br>

With Nginx installed, any traffic coming to your server will be handled by the web server.
<br>
Give it a try: type the IP address without the port in your browser!
<br>

### Configure
Default Nginx configuration is located in `/etc/nginx/sites-enabled/`
<br>

Create file with name of project 
```
sudo touch /etc/nginx/sites-available/nameProject
```
<br>

Create shortcuts for file 
```
sudo ln -s /etc/nginx/sites-available/nameProject /etc/nginx/sites-enabled/
```
<br>

Open and update file 
```
sudo vi /etc/nginx/sites-available/nameProject
```
<br>

Add in file
```
server { 
        
    listen 80; server_name MyIP; 
    root /home/user/nameProject/;
        
    location /static {
        alias /home/user/nameProject/nameProject/staticfiles/;
    }

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_redirect off;
        if (!-f $request_filename) {
            proxy_pass http://127.0.0.1:8000;
            break;
        }
    }
}
```

Close file
```
:wq
```
<br>

### Reload service Nginx
```
sudo service nginx reload
```
<br>
<br>
<br>



# GUNICORN
### Install 
```
pip3 install gunicorn
```
<br>

```
 cd MyApp
```
<br>

### Lanuch server 
```
gunicorn nameProject.wsgi:application
```
<br>
<br>
<br>



# SUPERVISOR
### Install 
```
sudo apt-get install supervisor
```
<br>

### Configure Supervisor
update file conf
```
sudo vi /etc/supervisor/conf.d/nameProject-gunicorn.conf
```
<br>

Add in file  
```	
[program:nameProject-gunicorn]
command = /home/user/nameProject/venv/bin/gunicorn nameProject.wsgi:application
user = user
directory = /home/user/nameProject/
autostart = true
autorestart = true
environment = ENV="PRODUCTION",SECRET_KEY="Newsecretkey"
```

### For Generate Secret Key in python terminal
```
import random, string
```

```
"".join([random.choice(string.printable) for _ in range(24)])
```
<br>
 
### Launch Supervisor
```
sudo supervisorctl reread
```

```
sudo supervisorctl update
```

```
sudo supervisorctl status
```
<br>
<br>
<br>

























