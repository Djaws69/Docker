
# Installer Docker

```
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install docker-ce
```

## Demarer Docker 

```
sudo systemctl start docker
```

```
sudo systemctl enable docker
```

```
sudo usermod -aG docker $(whoami)
```

```
docker run hello-world
```

## Docker Run 

```
azureuser@Test:/$ docker run --name web -d -v /home/azureuser/html/index.html:/usr/share/nginx/html/index.html -p 9999:80 nginx
22de3667605145cd75bd482733eae9e8b2c5d6656a2e0bfd6e8b185f195d579a
```

```
azureuser@Test:/$ curl 127.0.0.1:9999
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Accueil - Mon Site</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      background-color: #f4f4f4;
    }
    header {
      background-color: #333;
      color: #fff;
      padding: 20px;
      text-align: center;
    }
    nav {
      background-color: #444;
      overflow: hidden;
    }
    nav ul {
      list-style: none;
      margin: 0;
      padding: 0;
      display: flex;
      justify-content: center;
    }
    nav ul li {
      margin: 0;
    }
    nav ul li a {
      display: block;
      padding: 14px 20px;
      color: #fff;
      text-decoration: none;
    }
    nav ul li a:hover {
      background-color: #555;
    }
    section {
      padding: 20px;
    }
    footer {
      background-color: #333;
      color: #fff;
      text-align: center;
      padding: 10px;
      position: fixed;
      bottom: 0;
      width: 100%;
    }
  </style>
</head>
<body>
  <header>
    <h1>Bienvenue sur Mon Site</h1>
  </header>

  <nav>
    <ul>
      <li><a href="index.html">Accueil</a></li>
      <li><a href="about.html">À propos</a></li>
      <li><a href="contact.html">Contact</a></li>
    </ul>
  </nav>

  <section>
    <h2>Page d'Accueil</h2>
    <p>Ceci est un exemple de page d'accueil pour votre site web. Vous pouvez y ajouter du contenu, des images, et des liens vers d'autres pages.</p>
  </section>

  <footer>
    <p>&copy; 2025 Mon Site. Tous droits réservés.</p>
  </footer>
</body>
</html>
azureuser@Test:/$
```

## Custom un peu le lancement du conteneur

```
docker run -d --name meow -p 7777:7777 --memory=512M -v /etc/nginx/conf.d/nginx.conf:/etc/nginx/conf.d/nginx.conf -v /var/www/tp_docker:/var/www/tp_docker/index.html nginx
```

```
azureuser@Test:~$ curl 127.0.0.1:7777
<html>
<head>
  <meta charset="utf-8">
  <title>Mon site personnalisé</title>
</head>
<body>
  <h1>Bienvenue sur mon site personnalisé !</h1>
</body>
</html>
```

# Etape 2 

## Construire votre propre image

```
azureuser@Test:/imagine$ docker run --name myapache -p 8888:80 -d imagine
d5d2d68113e8e7442c4193156c91014bc235eb73bc5d6feaf9e1a17728841a40
azureuser@Test:/imagine$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                     NAMES
d5d2d68113e8   imagine   "apache2 -DFOREGROUND"   5 seconds ago   Up 5 seconds   0.0.0.0:8888->80/tcp, [::]:8888->80/tcp   myapache
azureuser@Test:/imagine$ curl localhost:8888
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Bienvenue sur mon Apache personnalisé</title>
</head>
<body>
  <h1>Hello depuis Docker !</h1>
  <p>Ceci est une page d'accueil personnalisée.</p>
</body>
</html>
azureuser@Test:/imagine$
```

```
FROM debian:latest

RUN apt-get update && apt-get upgrade -y &&     apt-get install -y apache2

COPY apache2.conf /etc/apache2/apache2.conf

COPY index.html /var/www/html/index.html

EXPOSE 80

CMD ["apache2", "-DFOREGROUND"]

RUN mkdir -p /etc/apache2/logs
```

# Etape 3 

## Installez un WikiJS en utilisant Docker

```
azureuser@Test:/$ docker-compose ps
  Name                 Command               State                         Ports
-------------------------------------------------------------------------------------------------------
wikijs      docker-entrypoint.sh node  ...   Up      0.0.0.0:3000->3000/tcp,:::3000->3000/tcp, 3443/tcp
wikijs_db   docker-entrypoint.sh postgres    Up      5432/tcp
azureuser@Test:/$
```

```
azureuser@Test:/$ curl http://40.81.240.84:3000/
<!DOCTYPE html><html><head><meta http-equiv="X-UA-Compatible" content="IE=edge"><meta charset="UTF-8"><meta name="viewport" content="user-scalable=yes, width=device-width, initial-scale=1, maximum-scale=5"><meta name="theme-color" content="#1976d2"><meta name="msapplication-TileColor" content="#1976d2"><meta name="msapplication-TileImage" content="/_assets/favicons/mstile-150x150.png"><title>Wiki.js Setup</title><link rel="apple-touch-icon" sizes="180x180" href="/_assets/favicons/apple-touch-icon.png"><link rel="icon" type="image/png" sizes="192x192" href="/_assets/favicons/android-chrome-192x192.png"><link rel="icon" type="image/png" sizes="32x32" href="/_assets/favicons/favicon-32x32.png"><link rel="icon" type="image/png" sizes="16x16" href="/_assets/favicons/favicon-16x16.png"><link rel="mask-icon" href="/_assets/favicons/safari-pinned-tab.svg" color="#1976d2"><link rel="manifest" href="/_assets/manifest.json"><script>var siteConfig = {"title":"Wiki.js"}</script><link type="text/css" rel="stylesheet" href="/_assets/css/setup.22871ffac1b643eed4d9.css"><script type="text/javascript" src="/_assets/js/runtime.js?1738531300"></script><script type="text/javascript" src="/_assets/js/setup.js?1738531300"></script></head><body><div id="root"><setup wiki-version="2.5.306"></setup></div></body></html>azureuser@Test:/$
```

## Meow app

```
azureuser@Test:/meow$ sudo git clone https://gitlab.com/it4lik/b2e-cloud-2024.git
Cloning into 'b2e-cloud-2024'...
remote: Enumerating objects: 49, done.
remote: Counting objects: 100% (49/49), done.
remote: Compressing objects: 100% (44/44), done.
remote: Total 49 (delta 13), reused 0 (delta 0), pack-reused 0 (from 0)
Receiving objects: 100% (49/49), 1.79 MiB | 7.95 MiB/s, done.
Resolving deltas: 100% (13/13), done.
azureuser@Test:/meow$ ls
b2e-cloud-2024
```

```
sudo nano Dockerfile
```

```
sudo nano docker-compose.yml
```

```
docker-compose up -d
```

```
azureuser@Test:/meow/b2e-cloud-2024/tp/1/python-app$ docker-compose ps
   Name                 Command               State                    Ports
----------------------------------------------------------------------------------------------
meow_app     python app.py                    Up      0.0.0.0:5555->5555/tcp,:::5555->5555/tcp
meow_redis   docker-entrypoint.sh redis ...   Up      0.0.0.0:6379->6379/tcp,:::6379->6379/tcp
```

```
docker-compose up --build -d
```

```
azureuser@Test:/meow/b2e-cloud-2024/tp/1/python-app$ curl 40.81.240.84:5555
 
StatusCode        : 200
StatusDescription : OK
Content           : <h1>Add key</h1>
                    <form action="/add" method = "POST">

                    Key:
                    <input type="text" name="key" >

                    Value:
                    <input type="text" name="value" >

                    <input type="submit" value="Submit">
                    </form>

                    <h1>Check key</h1>
                    ...
RawContent        : HTTP/1.1 200 OK
                    Connection: close
                    Content-Length: 340
                    Content-Type: text/html; charset=utf-8
                    Date: Sun, 16 Mar 2025 17:24:58 GMT
                    Server: Werkzeug/3.1.3 Python/3.9.21

                    <h1>Add key</h1>
                    <form act...
Forms             : {, }
Headers           : {[Connection, close], [Content-Length, 340], [Content-Type, text/html; charset=utf-8], [Date, Sun, 16 Mar 2025
                    17:24:58 GMT]...}
Images            : {}
InputFields       : {@{innerHTML=; innerText=; outerHTML=<INPUT name=key>; outerText=; tagName=INPUT; name=key}, @{innerHTML=;
                    innerText=; outerHTML=<INPUT name=value>; outerText=; tagName=INPUT; name=value}, @{innerHTML=; innerText=;
                    outerHTML=<INPUT type=submit value=Submit>; outerText=; tagName=INPUT; type=submit; value=Submit},
                    @{innerHTML=; innerText=; outerHTML=<INPUT name=key>; outerText=; tagName=INPUT; name=key}...}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 340
```

# Etape 4

## Prouvez que vous pouvez devenir root

```
azureuser@Test:/$ docker run --rm -it --privileged alpine sh
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
f18232174bc9: Already exists
Digest: sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c
Status: Downloaded newer image for alpine:latest
/ # ls
bin    dev    etc    home   lib    media  mnt    opt    proc   root   run    sbin   srv    sys    tmp    usr    var
/ # cat etc/shadow
root:*::0:::::
bin:!::0:::::
daemon:!::0:::::
lp:!::0:::::
sync:!::0:::::
shutdown:!::0:::::
halt:!::0:::::
mail:!::0:::::
news:!::0:::::
uucp:!::0:::::
cron:!::0:::::
ftp:!::0:::::
sshd:!::0:::::
games:!::0:::::
ntp:!::0:::::
guest:!::0:::::
nobody:!::0:::::
/ #
```

### 2. Scan de vuln
Il existe des outils dédiés au scan de vulnérabilités dans des images Docker.
C'est le cas de Trivy par exemple.
🌞 Utilisez Trivy

```
azureuser@Test:/$ trivy image nginx:latest
2025-03-17T15:10:18Z    INFO    Need to update DB
2025-03-17T15:10:18Z    INFO    Downloading DB...       repository="ghcr.io/aquasecurity/trivy-db:2"
61.00 MiB / 61.00 MiB [-----------------------------------------------------------------------------------------------------------] 100.00% 4.13 MiB p/s 15s
2025-03-17T15:10:35Z    INFO    Vulnerability scanning is enabled
2025-03-17T15:10:35Z    INFO    Secret scanning is enabled
2025-03-17T15:10:35Z    INFO    If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2025-03-17T15:10:35Z    INFO    Please see also https://aquasecurity.github.io/trivy/v0.52/docs/scanner/secret/#recommendation for faster secret detection
2025-03-17T15:10:48Z    INFO    Java DB Repository      repository=ghcr.io/aquasecurity/trivy-java-db:1
2025-03-17T15:10:48Z    INFO    Downloading the Java DB...
697.86 MiB / 697.86 MiB [-------------------------------------------------------------------------------------------------------] 100.00% 9.29 MiB p/s 1m15s
2025-03-17T15:12:05Z    INFO    The Java DB is cached for 3 days. If you want to update the database more frequently, the '--reset' flag clears the DB cache.
2025-03-17T15:12:06Z    INFO    Detected OS     family="debian" version="12.9"
2025-03-17T15:12:06Z    INFO    [debian] Detecting vulnerabilities...   os_version="12" pkg_num=149
2025-03-17T15:12:06Z    INFO    Number of language-specific files       num=0

nginx:latest (debian 12.9)

Total: 156 (UNKNOWN: 0, LOW: 99, MEDIUM: 42, HIGH: 13, CRITICAL: 2)
```

```
azureuser@Test:/$ trivy image requarks/wiki:latest
Report Summary

┌──────────────────────────────────────────────────────────────────────────────────┬──────────┬─────────────────┬─────────┐
│                                      Target                                      │   Type   │ Vulnerabilities │ Secrets │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ requarks/wiki:latest (alpine 3.21.2)                                             │  alpine  │       27        │    -    │
└──────────────────────────────────────────────────────────────────────────────────┴──────────┴─────────────────┴─────────┘
```

```
azureuser@Test:/$ trivy image postgres:13
Report Summary

┌────────────────────────────────────────┬──────────┬─────────────────┬─────────┐
│                 Target                 │   Type   │ Vulnerabilities │ Secrets │
├────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ postgres:13 (debian 12.9)              │  debian  │       152       │    -    │
└────────────────────────────────────────┴──────────┴─────────────────┴─────────┘
```

