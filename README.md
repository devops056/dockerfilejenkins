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

#### Step - 1 -Creat Dockerfile and Build Image, please find the below command, refer these snaps - (creating jenkins image, jenkins image created).
created Dockerfile as per uploded file and builded image using below command and also run it.
```
docker build -t myjenkins:v1 . (here"." means we are running this command from present directory of Dockerfile)
```

#### Step - 2 -Run that Image using below command, refer these snaps - (Jenkins docker run).
```
docker run -it -P -v /wwwroot/devops_task_2:/home/code/ -v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker --name myjenkins1 myjenkins:v1 
```
(Note: we have attached docker socket and configuration file of base os to perfrom any commnad from jenkins container, here this is pvc - /wwwroot/devops_task_2/ )

#### Step - 3 - Login in url of jenkins using below command to find exposed port, please find the below command, refer these snaps - (initial password, Exposed port, use login url and initial password).
```
docker ps
```
-By using base os ip to open jenkins(http://192.168.99.101:32769/)

-Use default password shown in this file (/root/.jenkins/secrets/initialAdminPassword) 

-Change the admin password then create below jobs

#### Step - 4 - Job-1 -Pull the code from GitHub when developers pushed to Github using poll SCM, please find the below code, refer these snaps - (Github plugin installation, Job-1-snap-1, Job-1-snap-2).
-First of all, install GitHub plugin in jenkins from manage jenkins.

-pull the code from GitHub and run below command to copy those files from jenkins workspace to that folder
```
if ls /home/ | grep code
then
	echo "Directory already present"
else
	sudo mkdir /home/code
fi

sudo rm -rf /home/code/*
sudo cp -rvf * /home/code/
```

#### Step - 5 - Job-2 -this job run if job1 build successfully -it will check code, run respective container(PHP or HTML), please find the below code, refer these snaps - (Job-2-snap-1, Job-2-snap-2, Github php code, PHP code running, Github html code, HTML code running).

```
if sudo ls /home/code/ | grep .php
then
	if sudo docker ps | grep phpos
	then
		echo "PHPOS is already running"
		sudo docker exec phpos rm -f /var/www/html/*.php
		sudo docker cp /home/code/*.php phpos:/var/www/html/
	else
		if sudo docker ps -a | grep phpos
		then
			echo "PHPOS is stopped"
			sudo docker start phpos
			sudo docker exec phpos rm -f /var/www/html/*.php
			sudo docker cp /home/code/*.php phpos:/var/www/html
		else
			sudo docker run -dit --name phpos -p 80:80 vimal13/apache-webserver-php
			sudo docker exec phpos rm -f /var/www/html/*.php
			sudo docker cp /home/code/*.php phpos:/var/www/html/
		fi
	fi
else
	echo "No PHP code available"
fi
```
```
if sudo ls /home/code/ | grep .html
then
	if sudo docker ps | grep htmlos
	then
		echo "HTMLOS is already running"
		sudo docker exec htmlos rm -f /var/www/html/*.html
		sudo docker cp /home/code/*.html htmlos:/var/www/html/
	else
		if sudo docker ps -a | grep htmlos
		then
			echo "HTMLOS is stopped"
			sudo docker start htmlos
			sudo docker exec htmlos rm -f /var/www/html/*.html
			sudo docker cp /home/code/*.html htmlos:/var/www/html/
		else
			sudo docker run -dit --name htmlos -p 8081:80 krushnakant241/mywebos:v1
			sudo docker exec htmlos rm -f /var/www/html/*.html
			sudo docker cp /home/code/*.html htmlos:/var/www/html/
		fi
	fi
else
	echo "No HTML code available"
fi
```

#### Step - 6 - Job-3 -this job run if job2 build successfully -it will test the code, it is working or not, refer these snaps - (Job-3-snap-1, Job-3-snap-2).

```
if sudo ls /home/code/ | grep index.php
then
	export status=$(curl -o /dev/null -s -w "%{http_code}" http://192.168.99.101/index.php)
	if [ $status -eq 200 ]
	then
		exit 0
	else
		echo "No PHP code found"
	exit 1
	fi
else
	if sudo ls /home/code/ | grep index.html
	then
		export status1=$(curl -o /dev/null -s -w "%{http_code}" http://192.168.99.101:8081/index.html)
		if [ $status1 -eq 200 ]
		then
			exit 0
		else
			echo "No HTML code found"
			exit 1
		fi
	fi
echo "No suitable code found"
exit 1
fi
```

#### Step - 7 - Job-4 -this job run if job3 build unsuccessful -it will send notification to developer, refer these snaps - (job-4, Code error-failed notification).
```
echo "There is a some error in code or no suitable code found, please refer the logs of job3"
exit 1
```

#### Step - 8 - Job-5 -this job check that container is running or not in every 1 minute, if not then run it again, refer these snaps - (Job-5-snap-1, Job-5-snap-2).

```
if  sudo docker ps | grep phpos
then
	echo "PHPOS is already running"
	exit 0
else
	if sudo docker ps -a | grep phpos
	then
		echo "PHPOS is stopped"
		sudo docker start phpos
	else
		sudo docker run -dit --name phpos -p 80:80 vimal13/apache-webserver-php
		sudo docker exec phpos rm -f /var/www/html/index.php
		sudo docker cp /home/code/*.php phpos:/var/www/html/
	fi
fi
```
```
if  sudo docker ps | grep htmlos
then
	echo "HTMLOS is already running"
	exit 0
else
	if sudo docker ps -a | grep htmlos
	then
		echo "HTMLOS is stopped"
		sudo docker start htmlos
	else
		sudo docker run -dit --name htmlos -p 8081:80 krushnakant241/mywebos:v1
		sudo docker exec htmlos rm -f /var/www/html/index.html
		sudo docker cp /home/code/*.html htmlos:/var/www/html/
	fi
fi
```

#### Please refer this snap for build pipeline view - (Build pipeline).
