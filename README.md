DevOps_Miniproject_2 (Dockerfile + Git + Jenkins + Testing)

Project Tasks:
1. Create container image that’s has Jenkins installed using dockerfile.
2. When we launch this image, it should automatically starts Jenkins service in the container.
3. Create a job chain of job1, job2, job3 and job4 using build pipeline plugin in Jenkins.
4. Job1 : Pull the Github repo automatically when some developers push repo to Github.
5. Job2 : By looking at the code or program file, Jenkins should automatically start the respective language interpreter install image container to deploy code (eg. If code is of PHP, then Jenkins should start the container that has PHP already installed).
6. Job3 : Test your app if it is working or not. If app is not working , then send email to developer with error messages.
7. Job4 : If container where app is running. fails due to any reson then this job should automatically start the container again.

Let's see step by step how to achieve this :
Step - 1 -Creat Dockerfile and Build Image
      -created Dockerfile as per uploded file and builded image using below command and also run it.
      -docker build -t myjenkins:v1 . (here"." means we are running this command from present directory of Dockerfile)
Step - 2 -Run that Image using below command (here we have attached,     
      -docker run -it --privileged -P -v /:/host --name myjenkins1 myjenkins:v1 (Note: "--privileged" this use to perfom any task in base os from container)
      -Use docker ps command to know exposed port and used base os ip to open jenkins(http://192.168.99.101:32768/)
      -Use default password shown in this file (/root/.jenkins/secrets/initialAdminPassword)
      -Change the admin password
Step - 3 - Job-1 -Pull the code from GitHub when developers pushed to Github,
      -First of all, install GitHub plugin in jenkins from manage jenkins.
      -pull the code from GitHub and run below command to add those files from jenkins workspace to that file
      -sudo cp -rvf * /host/files
Step - 4 - Job-2 -this job run if job1 build successfully -it will check code, run respective container(please find the below code)
      -chroot /host /bin/bash <<"EOT"
       export lan=$(ls -l /files | grep php| wc -l)
       if  [ $lan -gt 0 ]
       then
       sudo docker container rm -f webos
       sudo docker run -dit -p 8081:80 -v /files:/var/www/html --name webos vimal13/apache-webserver-php
       fi
       EOT
Step - 5 - Job-3 and 4 -this job run if job2 build successfully -it will check code if code is wrong then send notification to developer
      -export status=$(curl -o /dev/null -s -w "%{http_code}" 172.17.0.1:8082/index.php)
       if [ $status -eq 200 ]
       then exit 0
       else
       exit 1
       fi
Step - 6 - Job-5 -this job check that container is running or not every 1 minute.    
      -chroot /host /bin/bash <<"EOT"
       if  sudo docker ps | grep webos
       then
       exit 0
       else
       sudo docker container rm -f webos
       sudo docker run -dit -p 8081:80 -v /files:/var/www/html --name webos vimal13/apache-webserver-php
       fi
       EOT

