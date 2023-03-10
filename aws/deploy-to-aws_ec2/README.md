# Deploy to AWS

## Setup ec2 Instance for Deployment

Set `400` permissions to private key.

After logging in to ec2 instance as `ec2-user`, [add user to docker user group](https://stackoverflow.com/questions/47854463/docker-got-permission-denied-while-trying-to-connect-to-the-docker-daemon-socke) 
so you can run  docker commands without sudo.

```bash
sudo usermod -a -G docker ec2-user
grep docker /etc/group # should show => "docker:x:998:ec2-user"
newgrp docker
```

### Install `docker-compose`

```bash
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

docker-compose version
```

## Inbound/Outbound Rules in Security Group

Inbound
```
IPv4	HTTP	TCP	80	0.0.0.0/0
IPv4	HTTPS	TCP	443	0.0.0.0/0
IPv4	SSH	    TCP	22	<MY-IP>
```

Outbound
```
IPv4	"All traffic"	All	All	0.0.0.0/0
```

## Docker

Expose port 5000 in `Dockerfile`.

Connect host's port 80 to docker container's port 5000 `80:5000`.

```bash
docker-compose up -d
curl localhost
```

Open up the instance's public IPV4 address or public IPV4 DNS in browser.

`ec2-x-y-z-a.ap-south-1.compute.amazonaws.com `

## Resources

- [Deploying Docker Containers with AWS ec2 instance](https://medium.com/@chandupriya93/deploying-docker-containers-with-aws-ec2-instance-265038bba674)

## Deploy to ec2 with NGINX and GUNICORN

When you run the flask app in CLI, you will run the application development mode. Flask includes Werkzeug development server.

> The development server is not intended to be used on production systems. It was designed especially for development purposes and performs poorly under high load. (https://werkzeug.palletsprojects.com/en/0.14.x/serving/)

### NGINX

Install nginx.

```bash
sudo apt install nginx
# open up config file
sudo vim /etc/nginx/nginx.conf
```

Comment out the pages served by default.

```conf
   # include /etc/nginx/sites-enabled/*;
```

Add a reverse proxy to our server running in port 5000.

```conf
   http {
    ...

        server {
                listen 80;
                access_log /var/log/nginx/access.log;

                location / {
                        proxy_pass http://localhost:5000;
                }
        }
   }
```

Now `localhost` will point to our flask server.


### GUNICORN

Install `gunicorn`.

```bash
pip install gunicorn
```

Run flask app with gunicorn's WSGI server.

```bash
gunicorn --bind 127.0.0.1:5000 hello:app
```

## Troubleshooting

Your instance's public DNS name https://ec2-x-x-x-x.y-z-1.compute.amazonaws.com/ is not loading. Verify that you could use `curl` to check the ports within the docker container.

```bash
docker exec -it <flask-container-id> /bin/bash
...
$ curl localhost:5000  # or whichever port your app is running on
...
docker exec -it <nginx-container-id> /bin/bash
...
$ curl localhost  # check port 80 of nginx container
```

Have you added a TLS certificate? No? Then change `https` in the url to `http`.

