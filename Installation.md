step by step installation for Debian / Ubuntu with virtualenv.

source : https://about.okhin.fr/posts/Searx/ with some additions

# basic installation
```sh
sudo apt-get install build-essential gcc libxslt-dev python-dev
sudo useradd searx
cd /usr/local
sudo git clone https://github.com/asciimoo/searx.git
sudo chown searx:searx -R /usr/local/searx
sudo searx
cd /usr/local/searx
virtualenv searx-ve
. searx-ve/bin/activate
pip install -r requirements.txt
```

# configuration
```
cp engines.cfg_sample engines.cfg
sed -i -e "s/ultrasecretkey/`openssl rand -hex 16`/g" searx/settings.py
```

edit searx/settings.py and / or engines.cfg if necessary :

```python
port = 8888
secret_key = "ultrasecretkey" # change this!
debug = True
request_timeout = 5.0 # seconds
weights = {} # 'search_engine_name': float(weight) | default is 1.0
blacklist = [] # search engine blacklist
categories = {} # custom search engine categories
base_url = None # "https://your.domain.tld/" or None (to use request parameters)
```

# check
check everything is fine :
```
python searx/webapp.py
```

check [http://localhost:8888](http://localhost:8888)

if every works fine, disable debug option in searx/settings.py :
```
sed -i -e "s/debug = True/debug = False/g" settings.py
```

# uwsgi

```
sudo apt-get install uwsgi uwsgi-plugin-python
```

Create /etc/uwsgi/apps-available/searx.ini with :
```
[uwsgi]
# Who will run the code
uid = searx
gid = searx

# Number of workers
workers = 4

# The right granted on the created socket
chmod-socket = 666

# Plugin to use and interpretor config
single-interpreter = true
master = true
plugin = python

# Application base folder
base = /usr/local/searx

# Module to import
module = searx.webapp

# Virtualenv and python path
virtualenv = /usr/local/searx/searx-ve/
pythonpath = /usr/local/searx/
chdir = /usr/local/searx/searx/

# The variable holding flask application
callable = app
```

More sh scripts :
```sh
cd /etc/uwsgi/apps-enabled
ln -s ../apps-available/searx.ini
/etc/init.d/uwsgi restart
```

# web server
## with nginx
If nginx is not installed :
```sh
sudo apt-get install nginx
```

create /etc/nginx/sites-available/searx with :
```Nginx
server {
    listen 80;
    server_name searx.example.com;
    root /usr/local/searx

    location / {
            include uwsgi_params;
            uwsgi_pass unix:/run/uwsgi/app/searx/socket;
    }
}
```

## with apache 
**FIXME : not tested**

```sh
sudo apt-get install libapache2-mod-wsgi
sudo a2enmod mod-wsgi
sudo /etc/init.d/apache2 restart
```

```
<Location />
    Options FollowSymLinks Indexes
    SetHandler uwsgi-handler
    uWSGISocket /run/uwsgi/app/searx/socket
</Location>
```