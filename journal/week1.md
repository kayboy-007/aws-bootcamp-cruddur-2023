# Week 1 — App Containerization

## Homework Challenges

### Run the dockerfile CMD as an external script
 The frontend application had the following CMD line: 
 
 ```Dockerfile
 CMD ["npm", "start"]
 ```
 
 The line above instructs the container to start the npm after installation. To run this as an external script, I have created a bash script that starts npm. I ran into a challenge here as VI was previously installed in the container.
 
![See screenshot of error](assets/Week1/Backend%20Image%20running%20in%20Docker%20Desktop.jpg)
 
 
 So I had to run the following commands to install it by running the following commands.
 
 ```bash
apt-get update
apt-get install vim
 ```
 Once the editor was installed, I created my Bash script in the container by creating a cmd.sh file and adding the following commands to the file.
 
 ```bash
 #!/bin/bash
 npm start
 ```
 
After creating the script, I made it executable and ran the script using the following commands:
```bash
chmod +x cmd.sh
./cmd.sh
```
which then prompted me to host the app on another port since 3000 was already in use. A better way to test this would be to remove the CMD line from my docker file but there's no need to since I've seen my script work.

### Push and tag a image to DockerHub
I did this by integrating Docker hub in my Gitpod environment. Clicking the REGISTRIES tab provides an option to do this. Since I already had a docker hub account, I just provided my username and password and my registry was connected. 

![See screenshot of Docker Integration on Gitpod](assets/Week1/DockerHub%20Integration.jpg)


I pushed my docker images by running the following commands 

```bash
docker login -u kayoxx007
docker tag aws-bootcamp-cruddur-2023-frontend-react-js kayoxx007/aws-bootcamp-cruddur-2023-frontend-react-js
docker push kayoxx007/aws-bootcamp-cruddur-2023-frontend-react-js
docker tag aws-bootcamp-cruddur-2023-backend-flask kayoxx007/aws-bootcamp-cruddur-2023-backend-flask
docker push kayoxx007/aws-bootcamp-cruddur-2023-backend-flask
```

![See screenshot of docker images in my DockerHub repository](assets/Week1/Dockerhub%20Images.jpg)


### Use multi-stage building for a Dockerfile build

From my research, here's a multi-stage build for the frontend application. 

```
# Build stage
FROM node:16.18 as build
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

# Runtime stage
FROM node:16.18
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY package.json .
RUN npm install --only=test
EXPOSE 3000
CMD ["npm", "start"]

```



### Implement a healthcheck in the V3 Docker compose file
I was able to research healthcheck by following the Docker official documentation at [Docker Hub Compose file documentation](https://docs.docker.com/compose/compose-file/) . I added my healthcheck to the frontend block of the compose file.

```yaml
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    healthcheck:
      test: ["CMD", "curl", "-f", "https://localhost:3000"]
      interval: 30s
      timeout: 10s
      retries: 5
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js

# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur
```

### Learn how to install Docker on your localmachine and get the same containers running outside of Gitpod / Codespaces
I installed Docker Desktop on my windows machine by following the instructions on [Docker Desktop Installation Guide](https://docs.docker.com/desktop/install/windows-install/).
I Linked my Dockerhub registry by clicking the Dockerhub button on the page. After supplying my credentials, I was able to pull the images in my repository to my laptop.

![See screenshot of docker images in my DockerHub repository](assets/Week1/Cruddur%20images%20in%20Docker%20Desktop.jpg "Cruddur Images in my Dockerhub repository")
![See screenshot of docker images in my DockerHub repository](assets/Week1/Container%20Terminal%20in%20Docker%20Desktop.jpg)


After pulling the images, I proceeded to create containers out of them by clicking and following the prompt. There are options to specify the host ports and path to link to the container volume so I specified port 3000 for the frontend application. 

![See screenshot of docker images in my DockerHub repository](assets/Week1/Backend%20Image%20running%20in%20Docker%20Desktop.jpg)


I was able to browse the frontend image in my local browser 

![See screenshot of docker images in my DockerHub repository](assets/Week1/Cruddur%20frontend%20on%20laptop.jpg)



### Launch an EC2 instance that has docker installed, and pull a container to demonstrate you can run your own docker processes.
I launched an EC2 instance in my AWS account by doing the following:
1. I created a keypair for use in openssh.
2. I created a VPC in my region.
3. I created a subnet in the availability zone.
4. I configured my security group to allow port 22 traffic to my EC2 instance?
5. I created an Internet gateway and NAT gateway.
6. I launched a t2 micro EC2 instance using Ubuntu's 22.04 image and attached my previous resources.

![See screenshot of docker images in my DockerHub repository](assets/Week1/EC2%20Instance%20console.jpg)


I connected to the EC2 instance using the key I downloaded from the AWS console, then installed Docker by following the [Docker Installation on Ubuntu](https://docs.docker.com/engine/install/ubuntu/).

![See screenshot of docker images in my DockerHub repository](assets/Week1/Docker%20installation%20on%20EC2.jpg)

Since my image was already in my Dockerhub registry,  I just had to run the following commands:

```bash
sudo docker pull kayoxx007/aws-bootcamp-cruddur-2023-backend-flask:latest
```

![See screenshot of docker images in my DockerHub repository](assets/Week1/Docker%20image%20pull%20on%20EC2.jpg)


I ran the image by running the following command

```bash
sudo docker run --rm -p 4567:4567 -it kayoxx007/aws-bootcamp-cruddur-2023-backend-flask
```
![See screenshot of docker images in my DockerHub repository](assets/Week1/Running%20Image%20on%20EC2%20instance.jpg)


I was able to confirm that my container was running and that the port 4567 was active by running the *netstat -a* command

![See screenshot of docker images in my DockerHub repository](assets/Week1/Netstat%20on%20EC2.jpg)


### Noteable Errors encounted
1. I had an error when working on the frontend notifications. I observed my Notifications page was blank. I also observed the following errors in my container logs.

```
192.168.64.201 - - [22/Feb/2023 19:46:32] "GET /api/activities/notifications HTTP/1.1" 200 -
192.168.64.201 - - [22/Feb/2023 19:47:08] "GET /api/activities/Notifications HTTP/1.1" 500 -
Traceback (most recent call last):
  File "/usr/local/lib/python3.10/site-packages/flask/app.py", line 2551, in __call__
    return self.wsgi_app(environ, start_response)
  File "/usr/local/lib/python3.10/site-packages/flask/app.py", line 2531, in wsgi_app
    response = self.handle_exception(e)
  File "/usr/local/lib/python3.10/site-packages/flask_cors/extension.py", line 165, in wrapped_function
    return cors_after_request(app.make_response(f(*args, **kwargs)))
  File "/usr/local/lib/python3.10/site-packages/flask/app.py", line 2528, in wsgi_app
    response = self.full_dispatch_request()
  File "/usr/local/lib/python3.10/site-packages/flask/app.py", line 1825, in full_dispatch_request
    rv = self.handle_user_exception(e)
  File "/usr/local/lib/python3.10/site-packages/flask_cors/extension.py", line 165, in wrapped_function
    return cors_after_request(app.make_response(f(*args, **kwargs)))
  File "/usr/local/lib/python3.10/site-packages/flask/app.py", line 1823, in full_dispatch_request
    rv = self.dispatch_request()
  File "/usr/local/lib/python3.10/site-packages/flask/app.py", line 1799, in dispatch_request
    return self.ensure_sync(self.view_functions[rule.endpoint])(**view_args)
  File "/backend-flask/app.py", line 107, in data_show_activity
    data = ShowActivity.run(activity_uuid=activity_uuid)
```

A further dive into the code revealed there was a case issue in my **aws-bootcamp-cruddur-2023/frontend-react-js/src/pages/NotificationsFeedPage.js** as I had *const backend_url = `${process.env.REACT_APP_BACKEND_URL}/api/activities/Notifications`* as opposed to *const backend_url = `${process.env.REACT_APP_BACKEND_URL}/api/activities/notifications`* which is shown in the code block below:

```React
export default function NotificationsFeedPage() {
  const [activities, setActivities] = React.useState([]);
  const [popped, setPopped] = React.useState(false);
  const [poppedReply, setPoppedReply] = React.useState(false);
  const [replyActivity, setReplyActivity] = React.useState({});
  const [user, setUser] = React.useState(null);
  const dataFetchedRef = React.useRef(false);

  const loadData = async () => {
    try {
      const backend_url = `${process.env.REACT_APP_BACKEND_URL}/api/activities/notifications`
      const res = await fetch(backend_url, {
        method: "GET"
      });
      let resJson = await res.json();
      if (res.status === 200) {
        setActivities(resJson)
      } else {
        console.log(res)
      }
    } catch (err) {
      console.log(err);
    }
  };
```

2. I ran into some issues generating keypair for my EC2 instance. But after reviewing the documentation, I was able to resolve this.

































