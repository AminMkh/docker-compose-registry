# Private docker registry locally
## Create Auth

create auth:
htpasswd -bnBC 10 "" docker_password | tr -d ':'

Or use: https://hostingcanada.org/htpasswd-generator/
Mode: BCrypt

docker:$2y$10$R8uxK9dhjfvpvyxVdyoKquyCB2HkuNiQH7r9QxarIbwudQi8bzLvK

Add to auth/htpasswd


## add DNS entry
~~~
sudo vi /etc/hosts
127.0.0.1 myregistry.com
~~~

## Run it
~~~
cd docker-registry
docker-compose up
~~~

## Login
~~~
export USERNAME=docker
export PASSWORD=docker_password
echo $PASSWORD | docker login myregistry.com:5000 -u $USERNAME --password-stdin

# Login Succeeded
~~~

## Test a push
~~~
docker pull nginx
docker image tag nginx:latest myregistry.com:5000/nginx:latest
docker push myregistry.com:5000/nginx:latest
~~~

## K8s

### Create secret
~~~
export DOCKER_REGISTRY_SERVER=myregistry.com:5000
export DOCKER_USER=docker
export DOCKER_PASSWORD=docker_password
kubectl create secret docker-registry regcred-local\
 --docker-server=$DOCKER_REGISTRY_SERVER\
 --docker-username=$DOCKER_USER\
 --docker-password=$DOCKER_PASSWORD
~~~

### run pod
~~~
kubectl run mynginx --image myregistry.com:5000/nginx:latest
# pod/mynginx created
k describe pod mynginx
k delete pod mynginx
~~~
