# DevOps_Task_2 (Dockerfile + Git + Jenkins + Testing)

## Project Tasks:
1. Create container image that’s has Jenkins installed using dockerfile.
2. When we launch this image, it should automatically starts Jenkins service in the container.
3. Create a job chain of job1, job2, job3 and job4 using build pipeline plugin in Jenkins.
4. Job1 : Pull the Github repo automatically when some developers push repo to Github.
5. Job2 : By looking at the code or program file, Jenkins should automatically start the respective language interpreter install image container to deploy code (eg. If code is of PHP, then Jenkins should start the container that has PHP already installed).
6. Job3 : Test your app if it is working or not.
7. Job4 : If app is not working , then send email to developer with error messages.
8. Job5 : If container where app is running. fails due to any reson then this job should automatically start the container again.

## Let's see step by step how to achieve this:

#### Step - 1 -Creat Dockerfile and Build Image
created Dockerfile as per uploded file and builded image using below command and also run it.
```
docker build -t myjenkins:v1 . (here"." means we are running this command from present directory of Dockerfile)
```

#### Step - 2 -Run that Image using below command     
```
docker run -it --privileged -P -v /:/host --name myjenkins1 myjenkins:v1 
```
(Note: "--privileged" this use to perfom any task in base os from container)

#### Step - 3 - Login in url of jenkins using below command to find exposed port,
```
docker ps
```
-By using base os ip to open jenkins(http://192.168.99.101:32770/)

-Use default password shown in this file (/root/.jenkins/secrets/initialAdminPassword) 

-Change the admin password then create below jobs

#### Step - 4 - Job-1 -Pull the code from GitHub when developers pushed to Github,
-First of all, install GitHub plugin in jenkins from manage jenkins.

-pull the code from GitHub and run below command to add those files from jenkins workspace to that file
```
if ls /host | grep code
then
echo "Directory already present"
else
sudo mkdir /host/code
fi

sudo cp -rvf * /host/code
```

#### Step - 5 - Job-2 -this job run if job1 build successfully -it will check code, run respective container(please find the below code)
   
```
if docker ps | grep phpos
then
sudo docker rm -f phpos
fi

if ls /var/code/ | grep .php
then
sudo docker run -it --name phpos -p 80:80 vimal13/apache-webserver-php
sudo docker exec phpos rm -f /var/www/html/index.php
sudo docker cp /var/code/*.php phpos:/var/www/html/
else
echo "No PHP code"
fi

if docker ps | grep htmlos
then
sudo docker rm -f htmlos
fi

if ls /var/code/ | grep .html
then
sudo docker run -it --name htmlos -p 80:80 krushnakant241/mywebos:v1
sudo docker exec htmlos rm -f /var/www/html/index.html
sudo docker cp /var/code/*.html htmlos:/var/www/html/
else
echo "No HTML code"
fi
```

#### Step - 6 - Job-3 -this job run if job2 build successfully -it will test the code, it is working or not.

```
export status=$(curl -o /dev/null -s -w "%{http_code}" 192.168.99.100/index.php)
if [ $status -eq 200 ]
then 
exit 0
else
exit 1
fi

export status=$(curl -o /dev/null -s -w "%{http_code}" 192.168.99.100/index.html)
if [ $status -eq 200 ]
then 
exit 0
else
exit 1
fi
```
#### Step - 7 - Job-4 -this job run if job3 build unsuccessful -it will send notification to developer

Please refer this image - 

#### Step - 8 - Job-5 -this job check that container is running or not every 1 minute, if not then run it again

```
if  sudo docker ps | grep phpos
then
exit 0
else
sudo docker rm -f phpos
sudo docker run -it --name phpos -p 80:80 vimal13/apache-webserver-php
sudo docker exec phpos rm -f /var/www/html/index.php
sudo docker cp /var/code/*.php phpos:/var/www/html/
fi

if  sudo docker ps | grep htmlos
then
exit 0
else
sudo docker rm -f htmlos
sudo docker run -it --name htmlos -p 80:80 krushnakant241/mywebos:v1
sudo docker exec htmlos rm -f /var/www/html/index.html
sudo docker cp /var/code/*.html htmlos:/var/www/html/
fi
```
